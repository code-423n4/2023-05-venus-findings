# underlying token could be swept accidentally/intentionally for Two address token.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L524
Two address tokens exists in the blockchain. For example, Synthetix's ProxyERC20 contract is such a token which exists in many forms (sUSD, sBTC...). Tokens if such are used as underlying, the owner/governance could sweep them even if they are used as underlying by providing the other address to the sweepToken function. The only check in this function is that `address(token) != underlying`, which is irrelevant in the case of two address tokens.
Recommedation:
check that the balance of the underlying token remains the same before and after the sweep.

# Missing `deadline` for swap operations
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L265
Swap can be done with a bad price in riskfund which may not be able to payoff the bad debt.

if an authorised user makes a swap transaction with a low transaction fees this be could be pending for hours, days, or weeks until transaction fees gets low for miners to be interested with the transaction.

When the transaction get mined even though the user get the `amountOutMin` the price of the assets could have dropped drastically since the user made the swap.

`swapPoolsAssets` interact with AMM pools but do not have a deadline parameter, it's passing block.timestamp in `_swapAsset` to a pool, which means that whenever the miner decides to include the txn in a block, it will be valid at that time, since block.timestamp will be the current timestamp.

Recommedation:
Add a deadline line parameter to `swapPoolsAssets` that should be passed to `_swapAsset`.


# Users could be liquidated unfairly
`preLiquidateHook` checks if `Action.LIQUIDATE` is in paused state but does not check if `Action.REPAY` for the borrowed token is in paused state.
The `Action.REPAY` for a borrowed token could be in paused state where a user cannot repay his/her borrow and the `Action.LIQUIDATE` may not be in a paused state, in this case users who borrowed token would be liquidated unfairly since the cannot repay their borrow. 
## Recommendation:
Also check if `Action.REPAY` is also in paused state if the token market is not deprecated.

# check the price from `getUnderlyingPrice` for priceError.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L240
check the price returned for errors.
```solidity
uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
            address(vToken)
        );
```
# functions can be refactored.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L397
move `_ensureValidName(name)` to the first line of the function.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1215
can be refactored to `_ensureMaxLoops(marketsCount + 1)` and placed at `#L1207`

initialize `markets.length` at the start and use throughout the function.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L162
```solidity
uint256 marketsCount = markets.length;
```
# fix misleading comment.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L250
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L927
state the exact intent of the code in comments uint256 cannot be -1
```diff
- * @param repayAmount The amount to repay, or -1 for the full outstanding amount
+ * @param repayAmount The amount to repay, or type(uint256).max for the full outstanding amount 
```

