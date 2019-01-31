代码位置：beacon-chain/core/blocks/block_operations.go

func  verifyAttesterSlashing(slashing *pb.AttesterSlashing, verifySignatures bool) error 

> 验证attester罚没条件是否成立，即double vote和surround vote

```
func verifyAttesterSlashing(slashing *pb.AttesterSlashing, verifySignatures bool) error {
   slashableVote1 := slashing.SlashableVote_1
   slashableVote2 := slashing.SlashableVote_2
   slashableVoteData1Attestation := slashableVote1.Data
   slashableVoteData2Attestation := slashableVote2.Data

   if err := verifySlashableVote(slashableVote1, verifySignatures); err != nil {
      return fmt.Errorf("could not verify attester slashable vote data 1: %v", err)
   }
   if err := verifySlashableVote(slashableVote2, verifySignatures); err != nil {
      return fmt.Errorf("could not verify attester slashable vote data 2: %v", err)
   }

   // Inner attestation data structures for the votes should not be equal,
   // as that would mean both votes are the same and therefore no slashing
   // should occur.
   if reflect.DeepEqual(slashableVoteData1Attestation, slashableVoteData2Attestation) {
      return fmt.Errorf(
         "attester slashing inner slashable vote data attestation should not match: %v, %v",
         slashableVoteData1Attestation,
         slashableVoteData2Attestation,
      )
   }

   // Unless the following holds, the slashing is invalid:
   // (vote1.justified_slot < vote2.justified_slot) &&
   // (vote2.justified_slot + 1 == vote2.slot) &&
   // (vote2.slot < vote1.slot)  
   // 等价于vote1.justified_slot < vote2.justified_slot < vote2.slot < vote1.slot
   // OR
   // vote1.slot == vote2.slot
   justificationValidity :=
      (slashableVoteData1Attestation.JustifiedSlot < slashableVoteData2Attestation.JustifiedSlot) &&
         (slashableVoteData2Attestation.JustifiedSlot+1 == slashableVoteData2Attestation.Slot) &&
         (slashableVoteData2Attestation.Slot < slashableVoteData1Attestation.Slot)

   slotsEqual := slashableVoteData1Attestation.Slot == slashableVoteData2Attestation.Slot

   if !(justificationValidity || slotsEqual) {
      return fmt.Errorf(
         `
         Expected the following conditions to hold:
         (slashableVoteData1.JustifiedSlot <
         slashableVoteData2.JustifiedSlot) &&
         (slashableVoteData2.JustifiedSlot + 1
         == slashableVoteData1.Slot) &&
         (slashableVoteData2.Slot < slashableVoteData1.Slot)
         OR
         slashableVoteData1.Slot == slashableVoteData2.Slot

         Instead, received slashableVoteData1.JustifiedSlot %d,
         slashableVoteData2.JustifiedSlot %d
         and slashableVoteData1.Slot %d, slashableVoteData2.Slot %d
         `,
         slashableVoteData1Attestation.JustifiedSlot,
         slashableVoteData2Attestation.JustifiedSlot,
         slashableVoteData1Attestation.Slot,
         slashableVoteData2Attestation.Slot,
      )
   }
   return nil
}
```