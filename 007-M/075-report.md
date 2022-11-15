CRYP70

medium

# Proposers Can Burn a Large Amount of Gas When Submitting Proposals With Long Description Strings

## Summary
Proposals are used throughout the protocol when suggesting changes after being voted on. When submitting a new proposal, the `_proposal()` function takes in a `_description` of type `string` .  If the `proposalRefund` state variable is set, the protocol will issue a refund to the user and the smart contract itself will take the expense of the amount of gas spent to create the proposal. There is no check on the description length which may cause the contract to spend a large amount of gas. 

## Vulnerability Detail
Because the description length of a new proposal can be arbitrary, users can create proposals with very long description strings which can burn an arbitrary amount of gas. Consider the following simple contract which emits an event when a description string is submitted similar to that of the `Governance.sol` contract:

```solidity
pragma solidity 0.8.13;


contract TestGas {

    event GasUsed(string);

    function returnsOne(string memory gasDescription) external {
        emit GasUsed(gasDescription);
    }

}
```

If the user enters a `gasDescription` of `hello`, the amount of gas consumed is `28203`. However, if a a `gasDescription` of `hellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohello` is submitted, the gas usage is bumped up to `37133`. 


## Impact
- If gas prices become increasingly expensive in the future, users or the protocol team will have to take the hit of even more expensive fees to create a new proposal. 
- Because the `_description` is of arbitrary length, a malicious user might want to drain the protocol of Ether to perhaps damage the protocol's reputation and deny the right for other users to claim refunds on gas fees (or delay their refunds). 
- A proposer might have some malicious intent to want to emit some arbitrary text as an Ethereum event and simply exploit this system to do so.

## Code Snippet
https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol#L347-L350
https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol#L425-L436


## Tool used
Manual Review and Remix. 

## Recommendation
I recommend the following actions:
- Check for a certain proposal description length to encourage short and concise descriptions.
- Change the type of the description from strings to bytes. 
