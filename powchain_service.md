PowChain服务

要想成为`Ethereum Serenity`的`beacon chain`或`shard chain`的`validator`，需要向`ETH1.0`的充值合约地址`depositContractAddress`发送交易，触发`Deposit`事件。当达到`CHAIN_START_FULL_DEPOSIT_THRESHOLD`时，触发`ChainStart`事件。`Powchain`服务主要负责监听事件等逻辑。

代码位置

beacon-chain/powchain/service.go

Web3Service结构

```
type Web3Service struct {
   ctx                    context.Context
   cancel                 context.CancelFunc
   client                 Client
   headerChan             chan *gethTypes.Header
   logChan                chan gethTypes.Log
   endpoint               string
   depositContractAddress common.Address
   chainStartFeed         *event.Feed
   reader                 Reader
   logger                 bind.ContractFilterer
   blockNumber            *big.Int    // the latest ETH1.0 chain blockNumber.
   blockHash              common.Hash // the latest ETH1.0 chain blockHash.
   vrcCaller              *contracts.DepositContractCaller
   depositRoot            []byte
   depositTrie            *trieutil.DepositTrie
}
```

Web3ServiceConfig结构

```
type Web3ServiceConfig struct {
   Endpoint        string         // ws或ipc
   DepositContract common.Address // 位于ETH1.0上的充值合约地址
   Client          Client     
   Reader          Reader
   Logger          bind.ContractFilterer
   ContractBackend bind.ContractBackend
}
```

func (w *Web3Service) ProcessLog(VRClog gethTypes.Log)

> 处理来自ETH1.0的Deposit和ChainStart事件。

```
// ProcessLog is the main method which handles the processing of all
// logs from the deposit contract on the ETH1.0 chain.
func (w *Web3Service) ProcessLog(VRClog gethTypes.Log) {
   // Process logs according to their event signature.
   // 注意：Log结构中Topics字段的第0个索引为事件签名的摘要
   // 第一个if处理Deposit事件 Deposit(bytes32,bytes,bytes)
   if VRClog.Topics[0] == hashutil.Hash(depositEventSignature) {
   	  //来自ETH1.0的充值事件Deposit
      w.ProcessDepositLog(VRClog)
      return
   }

	// 第二个if处理ChainStart事件
	/*event ChainStart(
        bytes indexed receiptRoot,
        bytes time
    );
    */
   if VRClog.Topics[0] == hashutil.Hash(chainStartEventSignature) {
      w.ProcessChainStartLog(VRClog)
      return
   }

   log.Debugf("Log is not of a valid event signature %#x", VRClog.Topics[0])
}
```

func (w *Web3Service) ProcessDepositLog(VRClog gethTypes.Log)

> 处理Deposit事件

```
func (w *Web3Service) ProcessDepositLog(VRClog gethTypes.Log) {
   merkleRoot, depositData, MerkleTreeIndex, err := contracts.UnpackDepositLogData(VRClog.Data)
   if err != nil {
      log.Errorf("Could not unpack log %v", err)
      return
   }
   if err := w.saveInTrie(depositData, merkleRoot); err != nil {
      log.Errorf("Could not save in trie %v", err)
      return
   }
   depositInput, err := blocks.DecodeDepositInput(depositData)
   if err != nil {
      log.Errorf("Could not decode deposit input  %v", err)
      return
   }
   index := binary.BigEndian.Uint64(MerkleTreeIndex)
   log.WithFields(logrus.Fields{
      "publicKey":         depositInput.Pubkey,
      "merkle tree index": index,
   }).Info("Validator registered in VRC with public key and index")
}
```

func (w *Web3Service) ProcessChainStartLog(VRClog gethTypes.Log)

> 处理ChainStart事件

```
// ProcessChainStartLog processes the log which had been received from
// the ETH1.0 chain by trying to determine when to start the beacon chain.
func (w *Web3Service) ProcessChainStartLog(VRClog gethTypes.Log) {
   receiptRoot, timestampData, err := contracts.UnpackChainStartLogData(VRClog.Data)
   if err != nil {
      log.Errorf("Unable to unpack ChainStart log data %v", err)
      return
   }
   if w.depositTrie.Root() != receiptRoot {
      log.Errorf("Receipt root from log doesn't match the root saved in memory,"+
         " want %#x but got %#x", w.depositTrie.Root(), receiptRoot)
      return
   }

   timestamp := binary.BigEndian.Uint64(timestampData)
   //判断ChainStart事件中的时间戳和当前事件戳的大小
   if uint64(time.Now().Unix()) < timestamp {
      log.Errorf("Invalid timestamp from log expected %d > %d", time.Now().Unix(), timestamp)
   }

   chainStartTime := time.Unix(int64(timestamp), 0)
   log.WithFields(logrus.Fields{
      "ChainStartTime": chainStartTime,
   }).Info("Minimum Number of Validators Reached for beacon-chain to start")
   //发送chainStartFeed
   w.chainStartFeed.Send(chainStartTime)
}
```

func (w *Web3Service) run(done <-chan struct{})

> 这是整个Web3Service的核心函数。

```
// run subscribes to all the services for the ETH1.0 chain.
func (w *Web3Service) run(done <-chan struct{}) {
   if err := w.initDataFromVRC(); err != nil {
      log.Errorf("Unable to retrieve data from VRC %v", err)
      return
   }
	
   // 订阅ETH1.0最新的块头，ETH1.0有新块，就会发送
   headSub, err := w.reader.SubscribeNewHead(w.ctx, w.headerChan)
   if err != nil {
      log.Errorf("Unable to subscribe to incoming ETH1.0 chain headers: %v", err)
      return
   }
   // 实例化一个合约日志过滤
   query := ethereum.FilterQuery{
      Addresses: []common.Address{
         w.depositContractAddress, //用合约地址指定过滤的合约
      },
   }
   // 订阅ETH1.0合约地址为w.depositContractAddress的事件日志，ETH1.0有新日志，就会发送
   logSub, err := w.logger.SubscribeFilterLogs(w.ctx, query, w.logChan)
   if err != nil {
      log.Errorf("Unable to query logs from VRC: %v", err)
      return
   }
   if err := w.processPastLogs(query); err != nil {
      log.Errorf("Unable to process past logs %v", err)
      return
   }
   defer logSub.Unsubscribe()
   defer headSub.Unsubscribe()

   for {
      select {
      case <-done:
         log.Debug("ETH1.0 chain service context closed, exiting goroutine")
         return
      case <-headSub.Err():  // 订阅出错
         log.Debug("Unsubscribed to head events, exiting goroutine")
         return
      case <-logSub.Err():  // 订阅出错
         log.Debug("Unsubscribed to log events, exiting goroutine")
         return
      case header := <-w.headerChan:
         w.blockNumber = header.Number
         w.blockHash = header.Hash()
         log.WithFields(logrus.Fields{
            "blockNumber": w.blockNumber,
            "blockHash":   w.blockHash.Hex(),
         }).Debug("Latest web3 chain event")
      case VRClog := <-w.logChan:
         log.Info("Received deposit contract log")
         w.ProcessLog(VRClog) //处理Deposit和ChainStart事件
      }
   }
}
```

func (w *Web3Service) processPastLogs(query ethereum.FilterQuery) error

> 处理以前的日志【估计这是新加入beacon chain的node在启动时要做的。处理以前的Deposit和ChainStart事件后，同先前加入beacon chain的nodes处于同一起跑线，就直接走case VRClog := <- w.logChan分支。】

```
func (w *Web3Service) processPastLogs(query ethereum.FilterQuery) error {
   logs, err := w.logger.FilterLogs(w.ctx, query)
   if err != nil {
      return err
   }

   for _, log := range logs {
      w.ProcessLog(log)
   }
   return nil
}
```

func (w *Web3Service) HasChainStartLogOccurred() (bool, time.Time, error)

> 为什么是Log0呢，而不是Log1或Log2或Log3或Log4？

```
func (w *Web3Service) HasChainStartLogOccurred() (bool, time.Time, error) {
   query := ethereum.FilterQuery{
      Addresses: []common.Address{
         w.depositContractAddress,
      },
   }
   logs, err := w.logger.FilterLogs(w.ctx, query)
   if err != nil {
      return false, time.Now(), fmt.Errorf("could not filter deposit contract logs: %v", err)
   }
   for _, log := range logs {
      if log.Topics[0] == hashutil.Hash(chainStartEventSignature) {
         _, timestampData, err := contracts.UnpackChainStartLogData(log.Data)
         if err != nil {
            return false, time.Now(), fmt.Errorf("unable to unpack ChainStart log data %v", err)
         }
         timestamp := binary.BigEndian.Uint64(timestampData)
         if uint64(time.Now().Unix()) < timestamp {
            return false, time.Now(), fmt.Errorf(
               "invalid timestamp from log expected %d > %d",
               time.Now().Unix(),
               timestamp,
            )
         }
         return true, time.Unix(int64(timestamp), 0), nil
      }
   }
   return false, time.Now(), nil
}
```

beacon chain spec

[Ethereum 1.0 deposit contract](https://github.com/ethereum/eth2.0-specs/blob/master/specs/core/0_beacon-chain.md#ethereum-10-deposit-contract)

