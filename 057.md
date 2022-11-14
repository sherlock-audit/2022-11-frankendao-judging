WATCHPUG

medium

# Unbounded loop may cause `lockedWhileVotesCast()` to revet due to out-of-gas

## Summary

Under normal circumstances, the number of `activeProposals` can increase to a certain number which makes the loop inside `ockedWhileVotesCast()` becomes too expensive and exceed the block limit.

## Vulnerability Detail

The length of `activeProposals` may increase over time, and when the length is large enough (`> 200`), the two rather expensive external calls to `governance.getReceipt()` and get proposalDate (`governance.getProposalData(activeProposals[i]);`) may revert the whole transaction due to out of gas, making it impossible to `_delegate()` and `_unstake()`.

## Impact

The users may not be able to `unstake()` which constitutes a temporarily frozen of funds.

## Code Snippet

https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Staking.sol#L166-L174

## Tool used

Manual Review

## Recommendation

Consider adding an upper bound for the total count of `activeProposals`, and one must clear or wait for another proposal to end before adding a new one.