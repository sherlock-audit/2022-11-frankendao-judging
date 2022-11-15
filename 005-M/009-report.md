neumo

medium

# Veto function should decrease proposalsPassed (and possibly proposalsCreated)

## Summary
When a proposal that has passed is vetoed, the proposer still has the bonus of `proposalsCreated` and `proposalsPassed` related to the proposal. Both should be decremented because `veto` is intended to be used for malicious proposals.

## Vulnerability Detail
The function `getCommunityVotingPower` returns a bonus to users for voting, creating proposals and having them passed.
https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Staking.sol#L520-L541
The `veto` function is supposed to be used against malicious proposals (see comment in line 522):
https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol#L520-L534
So the malicius proposer should not keep a bonus based in this proposal's creation and/or passing, and the function should decrease both the values of `proposalsCreated` and `proposalsPassed`. 

## Impact
Malicious proposer keeps bonus after his proposal is vetoed.

## Code Snippet
N/A

## Tool used

Manual Review

## Recommendation
Put a check in the `veto` function that decreases the values of both  `proposalsCreated` and `proposalsPassed`.
```solidity
if(proposal.verified){
	--userCommunityScoreData[proposal.proposer].proposalsCreated;
	--totalCommunityScoreData.proposalsCreated;
}
if (state(_proposalId) == ProposalState.Queued){
	--userCommunityScoreData[proposal.proposer].proposalsPassed;
	--totalCommunityScoreData.proposalsPassed;
}
```
