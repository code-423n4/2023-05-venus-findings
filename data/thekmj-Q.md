## [L-01] Incorrect `blocksPerYear` constant

The current `blocksPerYear` is set at a constant value of `2102400`. This is equal to $60 * 60 * 24 * 365 / 15$, which assumes block time to be $15$ seconds.

However, since the event of The Merge, Ethereum has been upgraded to 2.0, [which block time is at a constant value of 12 seconds](https://github.com/ethereum/consensus-specs/blob/v0.11.1/specs/phase0/beacon-chain.md#time-parameters).

This will cause the interest rate to be different from actually intended. Considering changing this value to the appropriate one. 

## [L-02] Assets that has not entered market can still be liquidated

[This is a bug that has existed since the original Compound](https://github.com/compound-finance/compound-protocol/issues/38). 

In the function `preLiquidateHook()`, there is no check for whether the collateral asset has actually been entered by the user. Therefore, an asset that is a non-collateral can still be liquidated.

This bug will technically open up possibilities for unintended fund loss. However, it should not be very impactful, considering that entering markets only result in strictly stronger borrow power for a user. Nevertheless, we believe this should be considered an unintended behavior.

## [L-03] Unconventional usage of `_ensureMaxLoops` on external functions

The `_ensureMaxLoops()` function is meant to revert if any array length is longer than a predefined threshold.

However, there are a lot of functions which max loop check is only dependent on the user's input, and not on any storage variable, then there really is no effect on the loop check, except for wasting a small amount of gas for each operation, while blocking the user from performing longer list of orders (e.g. when calling `liquidateAccount()` from the Comptroller).

We recommend not performing the check at all, at least not on calldata arrays, as the responsibility of ensuring the calldata length should be the users', and the protocol should not be blocking the users' opportunities.

It is worth noting that this is also a point of centralization risk, as pointed out by issue **[M-01]** of the automated findings, as the admin can set the value to zero and block all operations.

## [L-04] Recommending extra input validation for `PoolRegistry.addMarket()`

The VToken, which is a fork of CToken from the Compound V2 protocol, is vulnerable to the well-known first depositor attack. From the code, the sponsors seem to be aware of the attack, and has implemented mint-on-behalf upon market creation, in order to mitigate the risk.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L320-L324

However, there is no check that `input.initialSupply` is larger than zero, or larger than a certain threshold. Since the first-depositor attack is a high risk attack that can result in direct fund loss for users, we recommend implementing the check for extra safety, at a negligibly small gas cost for the admin only.

## [N-01] Redundant condition check

The following `if` block performs a condition check, reverting if not satisfied:

```solidity 
function _reduceReservesFresh(uint256 reduceAmount) internal {
    //...
    // We fail gracefully unless market's block number equals current block number
    if (accrualBlockNumber != _getBlockNumber()) {
        revert ReduceReservesFreshCheck();
    }
    //...
}
```

However, the only entrypoint to this internal function is through `reduceReserves`, which accrues interest right before calling. Therefore this condition won't ever be satisfied.

Removing this check will also has the virtue of small gas-savings.

