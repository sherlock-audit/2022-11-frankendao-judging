WATCHPUG

medium

# `queue()` should increase `proposalsPassed` instead of `proposalsCreated`

## Summary

`proposalsCreated` will be increased in `verifyProposal()`, `queue()` should increase `proposalsPassed`.

## Vulnerability Detail

Based on the context, `queue()` should increase `userCommunityScoreData[proposal.proposer].proposalsPassed` and `totalCommunityScoreData.proposalsPassed` instead.

## Impact

Due to the presence of `setProposalsCreatedMultiplier()` and `setProposalsPassedMultiplier()`, the multiplier of both scores can be different, when that's the case, the proposer's voting power bonuses will be wrongly calculated because `proposalsPassed` is not correctly increased in `queue()`.

## Code Snippet

https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol#L467-L495

https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Staking.sol#L592-L601

## Tool used

Manual Review

## Recommendation

Change to:

```diff
    function queue(uint256 _proposalId) external {
        // Succeeded means we're past the endTime, yes votes outweigh no votes, and quorum threshold is met
        if(state(_proposalId) != ProposalState.Succeeded) revert InvalidStatus();
        
        Proposal storage proposal = proposals[_proposalId];

        // Set the ETA (time for execution) to the soonest time based on the Executor's delay
        uint256 eta = block.timestamp + executor.DELAY();
        proposal.eta = eta.toUint32();

        // Queue separate transactions for each action in the proposal
        uint numTargets = proposal.targets.length;
        for (uint256 i = 0; i < numTargets; i++) {
            executor.queueTransaction(proposal.targets[i], proposal.values[i], proposal.signatures[i], proposal.calldatas[i], eta);
        }

        // If a proposal is queued, we are ready to award the community voting power bonuses to the proposer
-        ++userCommunityScoreData[proposal.proposer].proposalsCreated;
+        ++userCommunityScoreData[proposal.proposer].proposalsPassed;

        // We don't need to check whether the proposer is accruing community voting power because
        // they needed that voting power to propose, and once they have an Active Proposal, their
        // tokens are locked from delegating and unstaking.
-        ++totalCommunityScoreData.proposalsCreated;
+        ++totalCommunityScoreData.proposalsPassed;
        
        // Remove the proposal from the Active Proposals array
        _removeFromActiveProposals(_proposalId);

        emit ProposalQueued(_proposalId, eta);
    }
```