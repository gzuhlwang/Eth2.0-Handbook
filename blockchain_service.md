blockchain服务

blockchain服务是beacon chain最重要的功能之一。它设计处理区块（例如，验证）和更新beacon chain状态（验证通过，应用状态转移函数更新beacon chain状态）。

代码位置

beacon-chain/blockchain/service.go

ChainService结构

> 负责管理整个POS 信标链。

```
type ChainService struct {
   ctx                context.Context
   cancel             context.CancelFunc
   beaconDB           *db.BeaconDB
   web3Service        *powchain.Web3Service
   incomingBlockFeed  *event.Feed
   incomingBlockChan  chan *pb.BeaconBlock
   genesisTimeChan    chan time.Time
   canonicalBlockFeed *event.Feed
   canonicalStateFeed *event.Feed
   genesisTime        time.Time
   enablePOWChain     bool
}
```

func (c *ChainService) Start()

```
func (c *ChainService) Start() {
   // 获取beacon chain状态
   beaconState, err := c.beaconDB.State()
   if err != nil {
      log.Fatalf("Could not fetch beacon state: %v", err)
   }
   // If the chain has already been initialized, simply start the block processing routine.
   if beaconState != nil {
      log.Info("Beacon chain data already exists, starting service")
      c.genesisTime = time.Unix(int64(beaconState.GenesisTime), 0)
      go c.blockProcessing()
   } else {
      log.Info("Waiting for ChainStart log from the Validator Deposit Contract to start the beacon chain...")
      /* 订阅c.genesisTimeChan
      * 发送该事件的地方在func (w *Web3Service) ProcessChainStartLog(gethTypes.Log)这个函数
      */
      subChainStart := c.web3Service.ChainStartFeed().Subscribe(c.genesisTimeChan)
      go func() {
         // 订阅以后，就监听该管道。没有消息，该协程会阻塞。
         genesisTime := <-c.genesisTimeChan
         // 用ChainStart事件中传进来的时间作为beacon chain的创始时间
         if err := c.initializeBeaconChain(genesisTime); err != nil {
            log.Fatalf("Could not initialize beacon chain: %v", err)
         }
         // 进入区块处理逻辑
         go c.blockProcessing()
         subChainStart.Unsubscribe()
      }()
   }
}
```

func (c *ChainService) blockProcessing()

```
func (c *ChainService) blockProcessing() {
   subBlock := c.incomingBlockFeed.Subscribe(c.incomingBlockChan)
   defer subBlock.Unsubscribe()
   for {
      select {
      case <-c.ctx.Done():
         log.Debug("Chain service context closed, exiting goroutine")
         return

      // Listen for a newly received incoming block from the feed. Blocks
      // can be received either from the sync service, the RPC service,
      // or via p2p.
      case block := <-c.incomingBlockChan:
         beaconState, err := c.beaconDB.State()
         if err != nil {
            log.Errorf("Unable to retrieve beacon state %v", err)
            continue
         }

         if block.Slot > beaconState.Slot {
         	// 预处理区块，即验证区块
            computedState, err := c.ReceiveBlock(block, beaconState)
            if err != nil {
               log.Errorf("Could not process received block: %v", err)
               continue
            }
            if err := c.ApplyForkChoiceRule(block, computedState); err != nil {
               log.Errorf("Could not update chain head: %v", err)
               continue
            }
         }
      }
   }
}
```