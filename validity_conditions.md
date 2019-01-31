```
func IsValidBlock(
   ctx context.Context,
   state *pb.BeaconState,
   block *pb.BeaconBlock,
   enablePOWChain bool,
   HasBlock func(hash [32]byte) bool,
   GetPOWBlock func(ctx context.Context, hash common.Hash) (*gethTypes.Block, error),
   genesisTime time.Time) error {

   // Pre-Processing Condition 1:
   // Check that the parent Block has been processed and saved.
   // 检查BeaconBlock中的ParentRootHash32字段，即beacon chain的父区块，是否在BeaconDB中
   parentRoot := bytesutil.ToBytes32(block.ParentRootHash32)
   parentBlock := HasBlock(parentRoot)
   if !parentBlock {
      return fmt.Errorf("unprocessed parent block as it is not saved in the db: %#x", parentRoot)
   }

   // Pre-Processing Condition 2:
   // The state is updated up to block.slot -1.
   // beacon chain上的区块高度用slot表示。
   if state.Slot != block.Slot-1 {
      return fmt.Errorf(
         "block slot is not valid %d as it is supposed to be %d", block.Slot, state.Slot+1)
   }
	// 判断是否启用POWChain
   if enablePOWChain {
      h := common.BytesToHash(state.LatestDepositRootHash32)
      powBlock, err := GetPOWBlock(ctx, h)
      if err != nil {
         return fmt.Errorf("unable to retrieve POW chain reference block %v", err)
      }

      // Pre-Processing Condition 3:
      // The block pointed to by the state in state.processed_pow_receipt_root has
      // been processed in the ETH 1.0 chain.
      // 
      if powBlock == nil {
         return fmt.Errorf("proof-of-Work chain reference in state does not exist %#x", state.LatestDepositRootHash32)
      }
   }

   // Pre-Processing Condition 4:
   // The node's local time is greater than or equal to
   // state.genesis_time + block.slot * SLOT_DURATION.
   // 判断BeaconBlock的时间戳是否合法
   if !IsSlotValid(block.Slot, genesisTime) {
      return fmt.Errorf("slot of block is too high: %d", block.Slot)
   }

   return nil
}
```