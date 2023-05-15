|     |                                                                                                                                                                                                       | Instances |
| :-- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------: |
| 01  | Don't use of `Block.Timestamp` can be manipulated.                                                                                                                                                    |         2 |
| 02  | More instances for [this issue](https://gist.github.com/CloudEllie/6639dbfd7dc1809a3baa28bb2895e1d9#n08-else-block-not-required): `else`-block not required.                                          |         4 |
| 03  | `if` statement on a single line, without opening a block with curly brackets. Best Practice.                                                                                                          |        77 |
| 04  | One more instance for [this issue](https://gist.github.com/CloudEllie/6639dbfd7dc1809a3baa28bb2895e1d9#n10-if-statement-can-be-converted-to-a-ternary): `if`-statement can be converted to a ternary. |         1 |
| 05  | According to the syntax rules, use => `mapping ( ` instead of => `mapping(` using spaces as keyword.                                                                                                   |        29 |

## [01] Don't use of `Block.Timestamp` can be manipulated.

_[Docs](https://solidity-by-example.org/hacks/block-timestamp-manipulation/) Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts._

There are 2 instances:

[Pool/PoolRegistry.sol#L401](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L401)

```java
VenusPool memory pool = VenusPool(name, msg.sender, comptroller, block.number, block.timestamp);
```

[RiskFund/RiskFund.sol#L265](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L265)

```java
block.timestamp
```

Recommended Mitigation Steps:

-   Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

-   Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

## [02] More instances for [this issue](https://gist.github.com/CloudEllie/6639dbfd7dc1809a3baa28bb2895e1d9#n08-else-block-not-required): `else`-block not required.

There are 4 more instances:

[Comptroller.sol#L501-L516](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L501-L516)

```java
        if (seizerContract == address(this)) {
            // If Comptroller is the seizer, just check if collateral's comptroller
            // is equal to the current address
            if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {
                revert ComptrollerMismatch();
            }
        } else {
            // If the seizer is not the Comptroller, check that the seizer is a
            // listed market, and that the markets' comptrollers match
            if (!markets[seizerContract].isListed) {
                revert MarketNotListed(seizerContract);
            }
            if (VToken(vTokenCollateral).comptroller() != VToken(seizerContract).comptroller()) {
                revert ComptrollerMismatch();
            }
        }
```

change to:

```java
        if (seizerContract == address(this)) {
            // If Comptroller is the seizer, just check if collateral's comptroller
            // is equal to the current address
            if (address(VToken(vTokenCollateral).comptroller()) != address(this))
                revert ComptrollerMismatch();
        }
        // If the seizer is not the Comptroller, check that the seizer is a
        // listed market, and that the markets' comptrollers match
        if (!markets[seizerContract].isListed) revert MarketNotListed(seizerContract);
        if (VToken(vTokenCollateral).comptroller() != VToken(seizerContract).comptroller())
                revert ComptrollerMismatch();
```

[Comptroller.sol#L1350-L1357](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1351-L1357)

```java
if (snapshot.weightedCollateral > borrowPlusEffects) {
                snapshot.liquidity = snapshot.weightedCollateral - borrowPlusEffects;
                snapshot.shortfall = 0;
            } else {
                snapshot.liquidity = 0;
                snapshot.shortfall = borrowPlusEffects - snapshot.weightedCollateral;
            }
```

change to:

```java
if (snapshot.weightedCollateral > borrowPlusEffects) {
                snapshot.liquidity = snapshot.weightedCollateral - borrowPlusEffects;
                snapshot.shortfall = 0;
            }
snapshot.liquidity = 0;
snapshot.shortfall = borrowPlusEffects - snapshot.weightedCollateral;
```

[VToken.sol#L1078-L1082](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1078-L1082)

```java
if (address(vTokenCollateral) == address(this)) {
            _seize(address(this), liquidator, borrower, seizeTokens);
        } else {
            vTokenCollateral.seize(liquidator, borrower, seizeTokens);
        }
```

change to:

```java
if (address(vTokenCollateral) == address(this))
            _seize(address(this), liquidator, borrower, seizeTokens);

vTokenCollateral.seize(liquidator, borrower, seizeTokens);
```

[Rewards/RewardsDistributor.sol#L222-L227](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L222-L227)

```java
if (rewardTokenSpeed == 0) {
            // release storage
            delete lastContributorBlock[contributor];
        } else {
            lastContributorBlock[contributor] = getBlockNumber();
        }
```

change to:

```java
                            // release storage
if (rewardTokenSpeed == 0) delete lastContributorBlock[contributor];

lastContributorBlock[contributor] = getBlockNumber();
```

Rrecommended Mitigations Steps:

-   See 'change to' above

## [03] `if` statement on a single line, without opening a block with curly brackets. Best Practice.

_It is possible to write the code that should be run `if` the condition evaluates to true in the same line of the `if` statement (instead of inside the `if` block)._

1. Increases readability
2. Shortens the overall SLOC

There are 77 instances:

[Comptroller.sol#L194-L196](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L194-L196)

```ts
if (amountOwed != 0) {
            revert NonzeroBorrowBalance();
        }
```

e.g change to:

```ts
if (amountOwed != 0) revert NonzeroBorrowBalance();
```

[Comptroller.sol#L204-L206](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L204-L206)

```ts
if (!marketToExit.accountMembership[msg.sender]) {
    return NO_ERROR;
}
```

If the line is too long, you can write like this:

```java
if (!marketToExit.accountMembership[msg.sender])
    return NO_ERROR;
```

[Comptroller.sol#L256-L258](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L256-L258)

```ts
if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L266-L268](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L266-L268)

```ts
if (nextTotalSupply > supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
```

[Comptroller.sol#L333-L335](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L333-L335)

```ts
if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L345-L347](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L345-L347)

```ts
if (oracle.getUnderlyingPrice(vToken) == 0) {
            revert PriceError(address(vToken));
        }
```

[Comptroller.sol#L354-L356](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L354-L356)

```ts
if (nextTotalBorrows > borrowCap) {
              revert BorrowCapExceeded(vToken, borrowCap);
          }
```

[Comptroller.sol#L367-L369](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L367-L369)

```ts
if (snapshot.shortfall > 0) {
            revert InsufficientLiquidity();
        }
```

[Comptroller.sol#L395-L397](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L395-L397)

```ts
if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L439-L444](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L439-L444)

```ts
if (!markets[vTokenBorrowed].isListed) {
            revert MarketNotListed(address(vTokenBorrowed));
        }
if (!markets[vTokenCollateral].isListed) {
            revert MarketNotListed(address(vTokenCollateral));
        }
```

[Comptroller.sol#L450-L452](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L450-L452)

```ts
if (repayAmount > borrowBalance) {
                revert TooMuchRepay();
            }
```

[Comptroller.sol#L459-L462](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L459-L462)

```ts
if (snapshot.totalCollateral <= minLiquidatableCollateral) {
            /* The liquidator should use either liquidateAccount or healAccount */
            revert MinimalCollateralViolated(minLiquidatableCollateral, snapshot.totalCollateral);

```

[Comptroller.sol#L464-L466](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L464-L466)

```ts
if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }
```

[Comptroller.sol#L470-L472](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L470-L472)

```ts
if (repayAmount > maxClose) {
            revert TooMuchRepay();
        }
```

[Comptroller.sol#L497-L499](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L497-L499)

```ts
if (!markets[vTokenCollateral].isListed) {
            revert MarketNotListed(vTokenCollateral);
        }
```

[Comptroller.sol#L591-L593](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L591-L593)

```ts
if (snapshot.totalCollateral > minLiquidatableCollateral) {
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }
```

[Comptroller.sol#L595-L597](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L595-L597)

```ts
if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }
```

[Comptroller.sol#L607-L609](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L607-L609)

```ts
if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
            revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
        }
```

[Comptroller.sol#L618-L620](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L618-L620)

```ts
if (tokens != 0) {
    market.seize(liquidator, user, tokens);
}
```

[Comptroller.sol#L622-L624](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L622-L624)

```ts
if (borrowBalance != 0) {
    market.healBorrow(liquidator, user, repaymentAmount);
}
```

[Comptroller.sol#L646-L649](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L646-L649)

```ts
if (snapshot.totalCollateral > minLiquidatableCollateral) {
            // You should use the regular vToken.liquidateBorrow(...) call
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }
```

[Comptroller.sol#L655-L659](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L655-L659)

```ts
if (collateralToSeize >= snapshot.totalCollateral) {
            // There is not enough collateral to seize. Use healBorrow to repay some part of the borrow
            // and record bad debt.
            revert InsufficientCollateral(collateralToSeize, snapshot.totalCollateral);
        }
```

[Comptroller.sol#L661-L663](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L661-L663)

```ts
if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }
```

[Comptroller.sol#L670-L675](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L670-L675)

```ts
if (!markets[address(orders[i].vTokenBorrowed)].isListed) {
                revert MarketNotListed(address(orders[i].vTokenBorrowed));
            }
if (!markets[address(orders[i].vTokenCollateral)].isListed) {
                revert MarketNotListed(address(orders[i].vTokenCollateral));
            }
```

[Comptroller.sol#L735-L737](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L735-L737)

```ts
if (!market.isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L740-L742](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L740-L742)

```ts
if (newCollateralFactorMantissa > collateralFactorMaxMantissa) {
            revert InvalidCollateralFactor();
        }
```

[Comptroller.sol#L745-L747](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L745-L747)

```ts
if (newLiquidationThresholdMantissa > mantissaOne) {
            revert InvalidLiquidationThreshold();
        }
```

[Comptroller.sol#L750-L752](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L750-L752)

```ts
if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
            revert InvalidLiquidationThreshold();
        }
```

[Comptroller.sol#L755-L757](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755-L757)

```ts
if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
            revert PriceError(address(vToken));
        }
```

[Comptroller.sol#L804-L806](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L804-L806)

```ts
if (markets[address(vToken)].isListed) {
            revert MarketAlreadyListed(address(vToken));
        }
```

[Comptroller.sol#L1180](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1180-L1182)

```ts
if (!marketToJoin.isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L1184-L1187](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1184-L1187)

```ts
if (marketToJoin.accountMembership[borrower]) {
    // already joined
    return;
}
```

[Comptroller.sol#L1209-L1211](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1209-L1211)

```ts
if (allMarkets[i] == VToken(vToken)) {
                revert MarketAlreadyListed(vToken);
            }
```

[Comptroller.sol#L1245-L1247](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1245-L1247)

```ts
if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }
```

[Comptroller.sol#L1250-L1252](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1250-L1252)

```ts
if (!markets[vToken].accountMembership[redeemer]) {
    return;
}
```

[Comptroller.sol#L1262-L1264](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1262-L1264)

```ts
if (snapshot.shortfall > 0) {
            revert InsufficientLiquidity();
        }
```

[Comptroller.sol#L1370-L1372](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1370-L1372)

```ts
if (oraclePriceMantissa == 0) {
            revert PriceError(address(asset));
        }
```

[Comptroller.sol#L1413-L1415](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1413-L1415)

```ts
if (err != 0) {
            revert SnapshotError(address(vToken), user);
        }
```

[Comptroller.sol#L1422-L1424](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1422-L1424)

```ts
if (msg.sender != expectedSender) {
            revert UnexpectedSender(expectedSender, msg.sender);
        }
```

[Comptroller.sol#L1431-L1433](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1431-L1433)

```ts
if (actionPaused(market, action)) {
            revert ActionPaused(market, action);
        }
```

[VToken.sol#L313-L315](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L313-L315)

```ts
if (newProtocolSeizeShareMantissa_ + 1e18 > liquidationIncentive) {
            revert ProtocolSeizeShareTooBig();
        }

```

[VToken.sol#L395-L397](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L395-L397)

```ts
if (msg.sender != address(comptroller)) {
            revert HealBorrowUnauthorized();
        }
```

[VToken.sol#L456-L458](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L456-L458)

```ts
if (msg.sender != address(comptroller)) {
            revert ForceLiquidateBorrowUnauthorized();
        }
```

[VToken.sol#L684-L686](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L684-L686)

```ts
if (accrualBlockNumberPrior == currentBlockNumber) {
    return NO_ERROR;
}
```

[VToken.sol#L752-L754](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L752-L754)

```ts
if (accrualBlockNumber != _getBlockNumber()) {
            revert MintFreshnessCheck();
        }
```

[VToken.sol#L837-L839](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837-L839)

```ts
if (redeemTokens == 0 && redeemAmount > 0) {
    revert('redeemTokens zero');
}
```

[VToken.sol#L845-L847](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L845-L847)

```java
if (_getCashPrior() - totalReserves < redeemAmount) {
            revert RedeemTransferOutNotPossible();
        }
```

[VToken.sol#L882-L889](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L882-L889)

```ts
if (accrualBlockNumber != _getBlockNumber()) {
            revert BorrowFreshnessCheck();
        }

        /* Fail gracefully if protocol has insufficient underlying cash */
if (_getCashPrior() - totalReserves < borrowAmount) {
            revert BorrowCashNotAvailable();
        }
```

[VToken.sol#L939-L941](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L939-L941)

```ts
if (accrualBlockNumber != _getBlockNumber()) {
            revert RepayBorrowFreshnessCheck();
        }
```

[VToken.sol#L999-L1002](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L999-L1002)

```ts
if (error != NO_ERROR) {
            // accrueInterest emits logs on errors, but we still want to log the fact that an attempted liquidation failed
            revert LiquidateAccrueCollateralInterestFailed(error);
        }
```

[VToken.sol#L1035-L1057](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1035-L1057)

```ts
/* Verify market's block number equals current block number */
if (accrualBlockNumber != _getBlockNumber()) {
            revert LiquidateFreshnessCheck();
        }

        /* Verify vTokenCollateral market's block number equals current block number */
if (vTokenCollateral.accrualBlockNumber() != _getBlockNumber()) {
            revert LiquidateCollateralFreshnessCheck();
        }

        /* Fail if borrower = liquidator */
if (borrower == liquidator) {
            revert LiquidateLiquidatorIsBorrower();
        }

        /* Fail if repayAmount = 0 */
if (repayAmount == 0) {
            revert LiquidateCloseAmountIsZero();
        }

        /* Fail if repayAmount = -1 */
if (repayAmount == type(uint256).max) {
            revert LiquidateCloseAmountIsUintMax();
        }
```

[VToken.sol#L1107-L1109](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1107-L1109)

```java
if (borrower == liquidator) {
            revert LiquidateSeizeLiquidatorIsBorrower();
        }
```

[VToken.sol#L1161-L1163](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1161-L1163)

```java
if (newReserveFactorMantissa > reserveFactorMaxMantissa) {
            revert SetReserveFactorBoundsCheck();
        }
```

[VToken.sol#L1183-L1185](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1183-L1185)

```java
if (accrualBlockNumber != _getBlockNumber()) {
            revert AddReservesFactorFreshCheck(actualAddAmount);
        }
```

[VToken.sol#L1205-L1217](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1205-L1217)

```java
if (accrualBlockNumber != _getBlockNumber()) {
            revert ReduceReservesFreshCheck();
        }

        // Fail gracefully if protocol has insufficient underlying cash
if (_getCashPrior() < reduceAmount) {
            revert ReduceReservesCashNotAvailable();
        }

        // Check reduceAmount ≤ reserves[n] (totalReserves)
if (reduceAmount > totalReserves) {
            revert ReduceReservesCashValidation();
        }
```

[VToken.sol#L1248-L1250](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1248-L1250)

```java
if (accrualBlockNumber != _getBlockNumber()) {
            revert SetInterestRateModelFreshCheck();
        }
```

[VToken.sol#L1307-L1309](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1307-L1309)

```java
if (src == dst) {
            revert TransferNotAllowed();
        }
```

[VToken.sol#L1331-L1333](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1331-L1333)

```java
if (startingAllowance != type(uint256).max) {
            transferAllowances[src][spender] = allowanceNew;
        }
```

[VToken.sol#L1408-L1410](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1408-L1410)

```java
if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

[VToken.sol#L1446-L1448](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1446-L1448)

```java
if (borrowSnapshot.principal == 0) {
            return 0;
        }
```

[RewardsDistributor.sol#L134-L142](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L134-L142)

```java
if (supplyState.index == 0) {
            // Initialize supply state index with default value
            supplyState.index = rewardTokenInitialIndex;
        }
if (borrowState.index == 0) {
            // Initialize borrow state index with default value
            borrowState.index = rewardTokenInitialIndex;
        }
```

[Pool/PoolRegistry.sol#L422-L424](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422-L424)

```java
if (address(shortfall_) == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

[Pool/PoolRegistry.sol#L431-L433](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L431-L433)

```java
if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

[BaseJumpRateModelV2.sol#L97-L99](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L97-L99)

```java
if (!isAllowedToCall) {
            revert Unauthorized(msg.sender, address(this), signature);
        }
```

[BaseJumpRateModelV2.sol#L137-L139](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L137-L139)

```java
if (borrows == 0) {
            return 0;
        }
```

[WhitePaperInterestRateModel.sol#L92-L94](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L92-L94)

```java
if (borrows == 0) {
            return 0;
        }
```

[MaxLoopsLimitHelper.sol#L40-L42](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L40-L42)

```java
if (len > maxLoopsLimit) {
            revert MaxLoopsLimitExceeded(maxLoopsLimit, len);
        }
```

Recommended Mitigations Steps:

-   This is accomplished with a if statement on a single line, without opening a block with curly brackets.

## [04] One more instance for [this issue](https://gist.github.com/CloudEllie/6639dbfd7dc1809a3baa28bb2895e1d9#n10-if-statement-can-be-converted-to-a-ternary): `if`-statement can be converted to a ternary.

There an instance:

[Shortfall/Shortfall.sol#L239-L245](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L239-L245)

```java
        uint256 riskFundBidAmount;

        if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
            riskFundBidAmount = auction.seizedRiskFund;
        } else {
            riskFundBidAmount = (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS;
        }
```

change to:

```java
        uint256 riskFundBidAmount = auction.auctionType == AuctionType.LARGE_POOL_DEBT
                                ? auction.seizedRiskFund
                                : (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS;
```

## [05] According to the syntax rules, use `=> mapping ( ` instead of `=> mapping(` using spaces as keyword.

There are 29 instances:

[Shortfall/Shortfall.sol#L44](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L44)

```ts
- mapping(VToken => uint256) marketDebt;
+ mapping ( VToken => uint256 ) marketDebt;
```

[Shortfall/Shortfall.sol#L72](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L72)

```ts
- mapping(address => Auction) public auctions;
+ mapping ( address => Auction ) public auctions;
```

[Rewards/RewardsDistributor.sol#L23](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L23)

```ts
- mapping(address => RewardToken) public rewardTokenSupplyState;
+ mapping ( address => RewardToken ) public rewardTokenSupplyState;
```

[Rewards/RewardsDistributor.sol#L26](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L26)

```ts
- mapping(address => mapping(address => uint256)) public rewardTokenSupplierIndex;
+ mapping ( address => mapping ( address => uint256 )) public rewardTokenSupplierIndex;
```

[Rewards/RewardsDistributor.sol#L32](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L32)

```ts
- mapping(address => uint256) public rewardTokenAccrued;
+ mapping ( address => uint256 ) public rewardTokenAccrued;
```

[Rewards/RewardsDistributor.sol#L35](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L35)

```ts
- mapping(address => uint256) public rewardTokenBorrowSpeeds;
+ mapping ( address => uint256 ) public rewardTokenBorrowSpeeds;
```

[Rewards/RewardsDistributor.sol#L38](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L38)

```ts
- mapping(address => uint256) public rewardTokenSupplySpeeds;
+ mapping ( address => uint256 ) public rewardTokenSupplySpeeds;
```

[Rewards/RewardsDistributor.sol#L41](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L41)

```ts
- mapping(address => RewardToken) public rewardTokenBorrowState;
+ mapping ( address => RewardToken ) public rewardTokenBorrowState;
```

[Rewards/RewardsDistributor.sol#L44](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L44)

```ts
- mapping(address => uint256) public rewardTokenContributorSpeeds;
+ mapping ( address => uint256 ) public rewardTokenContributorSpeeds;
```

[Rewards/RewardsDistributor.sol#L47](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L47)

```ts
- mapping(address => uint256) public lastContributorBlock;
+ mapping ( address => uint256 ) public lastContributorBlock;
```

[Rewards/RewardsDistributor.sol#L50](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L50)

```ts
- mapping(address => mapping(address => uint256)) public rewardTokenBorrowerIndex;
+ mapping (address => mapping ( address => uint256 )) public rewardTokenBorrowerIndex;
```

[Pool/PoolRegistry.sol#L83](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L83)

```ts
- mapping(address => VenusPoolMetaData) public metadata;
+ mapping ( address => VenusPoolMetaData ) public metadata;
```

[Pool/PoolRegistry.sol#L88](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L88)

```ts
- mapping(uint256 => address) private _poolsByID;
+ mapping ( uint256 => address ) private _poolsByID;
```

[Pool/PoolRegistry.sol#L98](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L98)

```ts
- mapping(address => VenusPool) private _poolByComptroller;
+ mapping ( address => VenusPool ) private _poolByComptroller;
```

[Pool/PoolRegistry.sol#L103](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L103)

```ts
- mapping(address => mapping(address => address)) private _vTokens;
+ mapping ( address => mapping(address => address )) private _vTokens;
```

[Pool/PoolRegistry.sol#L108](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L108)

```ts
- mapping(address => address[]) private _supportedPools;
+ mapping (address => address[] ) private _supportedPools;
```

[RiskFund/RiskFund.sol#L35](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L35)

```ts
- mapping(address => uint256) public poolReserves;
+ mapping ( address => uint256 ) public poolReserves;
```

[VTokenInterfaces.sol#L107](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L107)

```ts
- mapping(address => uint256) internal accountTokens;
+ mapping(address => uint256) internal accountTokens;
```

[VTokenInterfaces.sol#L110](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L110)

```ts
- mapping(address => mapping(address => uint256)) internal transferAllowances;
+ mapping(address => mapping(address => uint256)) internal transferAllowances;
```

[VTokenInterfaces.sol#L113](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L113)

```ts
- mapping(address => BorrowSnapshot) internal accountBorrows;
+ mapping (address => BorrowSnapshot ) internal accountBorrows;
```

[ComptrollerStorage.sol#L41](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L41)

```ts
- mapping(address => bool) accountMembership;
+ mapping ( address => bool ) accountMembership;
```

[ComptrollerStorage.sol#L74](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L74)

```ts
- mapping(address => VToken[]) public accountAssets;
+ mapping (address => VToken[] ) public accountAssets;
```

[ComptrollerStorage.sol#L80](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L80)

```ts
- mapping(address => Market) public markets;
+ mapping (address => Market ) public markets;
```

[ComptrollerStorage.sol#L86](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L86)

```ts
- mapping(address => uint256) public borrowCaps;
+ mapping ( address => uint256 ) public borrowCaps;
```

[ComptrollerStorage.sol#L92](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L92)

```ts
- mapping(address => uint256) public supplyCaps;
+ mapping (address => uint256 ) public supplyCaps;
```

[ComptrollerStorage.sol#L95](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L95)

```ts
- mapping(address => mapping(Action => bool)) internal _actionPaused;
+ mapping (address => mapping ( Action => bool )) internal _actionPaused;
```

[ComptrollerStorage.sol#L101](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L101)

```ts
- mapping(address => bool) internal rewardsDistributorExists;
+ mapping (address => bool ) internal rewardsDistributorExists;
```

[RiskFund/ReserveHelpers.sol#L13](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L13)

```ts
- mapping(address => uint256) internal assetsReserves;
+ mapping (address => uint256 ) internal assetsReserves;
```

[RiskFund/ReserveHelpers.sol#L17](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L17)

```ts
- mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;
+ mapping (address => mapping ( address => uint256 )) internal poolsAssetsReserves;
```

Recommended Mitigations Steps:

-   follow the syntax
