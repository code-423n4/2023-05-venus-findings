### Non Critical Issues

| Number  | Issue                         | Instances |
| :-----: | :---------------------------- | :-------: |
| [NC-01] | Remove unnecessary comments   |     3     |
| [NC-02] | Use consistent comment syntax |    60     |
| [NC-03] | Use battle-tested code        |     1     |

#### [NC-01] Remove unnecessary comments

There are some comments in the code that are nearly identical to the line of code that follows it. These comments add no value, add unnecessary clutter, and increase maintenance costs. Consider removing these comments.

```solidity
File: contracts/VToken.sol

817:    /* If redeemTokensIn > 0: */
818:    if (redeemTokensIn > 0) {
```

```solidity
File: contracts/VToken.sol

1261:    // Emit NewMarketInterestRateModel(oldInterestRateModel, newInterestRateModel)
1262:    emit NewMarketInterestRateModel(oldInterestRateModel, newInterestRateModel);

1146:    // Emit NewComptroller(oldComptroller, newComptroller)
1147:    emit NewComptroller(oldComptroller, newComptroller);
```

#### [NC-02] Use consistent comment syntax

For single line comments, there is a mix of `/* ... */` and `//` used. There is no right or wrong way to write a single line comment, but consistency is an important factor in code readability and maintenance. Consider switching to one style of comment or the other.

For example, to switch all single line comments starting with `/*` to start with `//`, the following lines would need to be updated.

```solidity
File: contracts\Comptroller.sol:

190:         /* Get sender tokensHeld and amountOwed underlying from the vToken */

193:         /* Fail if the sender has a borrow balance */

198:         /* Fail if the sender is not permitted to redeem all of their tokens */

203:         /* Return true if the sender is not already ‘in’ the market */

208:         /* Set vToken account membership to false */

211:         /* Delete vToken from the account’s list of assets */

448:         /* Allow accounts to be liquidated if the market is deprecated or it is a forced liquidation */

456:         /* The borrower must have shortfall and collateral > threshold in order to be liquidatable */

460:             /* The liquidator should use either liquidateAccount or healAccount */

468:         /* The liquidator may not repay more than what is allowed by the closeFactor */

1089:         /* Read oracle prices for borrowed and collateral markets */

1249:         /* If the redeemer is not 'in' the market, then we can bypass the liquidity check */

1254:         /* Otherwise, perform a hypothetical liquidity check to guard against shortfall */
```

```solidity
File: contracts\JumpRateModelV2.sol:

20:     /* solhint-disable-next-line no-empty-blocks */
```

```solidity
File: contracts\VToken.sol:

679:         /* Remember the initial block number */

683:         /* Short-circuit accumulating 0 interest */

688:         /* Read the previous values out of storage */

694:         /* Calculate the current borrow interest rate */

698:         /* Calculate the number of blocks elapsed since the last accrual */

724:         /* We write the previously calculated values into storage */

730:         /* We emit an AccrueInterest event */

748:         /* Fail if mint not allowed */

751:         /* Verify market's block number equals current block number */

788:         /* We emit a Mint event, and a Transfer event */

807:         /* Verify market's block number equals current block number */

812:         /* exchangeRate = invoke Exchange Rate Stored() */

817:         /* If redeemTokensIn > 0: */

841:         /* Fail if redeem not allowed */

844:         /* Fail gracefully if protocol has insufficient cash */

868:         /* We emit a Transfer event, and a Redeem event */

878:         /* Fail if borrow not allowed */

881:         /* Verify market's block number equals current block number */

886:         /* Fail gracefully if protocol has insufficient underlying cash */

919:         /* We emit a Borrow event */

935:         /* Fail if repayBorrow not allowed */

938:         /* Verify market's block number equals current block number */

943:         /* We fetch the amount the borrower owes, with accumulated interest */

968:         /* We write the previously calculated values into storage */

973:         /* We emit a RepayBorrow event */

1025:         /* Fail if liquidate not allowed */

1034:         /* Verify market's block number equals current block number */

1039:         /* Verify vTokenCollateral market's block number equals current block number */

1044:         /* Fail if borrower = liquidator */

1049:         /* Fail if repayAmount = 0 */

1054:         /* Fail if repayAmount = -1 */

1059:         /* Fail if repayBorrow fails */

1066:         /* We calculate the number of collateral tokens that will be seized */

1074:         /* Revert if borrower collateral token balance < seizeTokens */

1084:         /* We emit a LiquidateBorrow event */

1103:         /* Fail if seize not allowed */

1106:         /* Fail if borrower = liquidator */

1126:         /* We write the calculated values into storage */

1132:         /* Emit a Transfer event */

1303:         /* Fail if transfer not allowed */

1306:         /* Do not allow self-transfers */

1311:         /* Get the allowance, infinite for the account owner */

1319:         /* Do the calculations, checking for {under,over}flow */

1330:         /* Eat some of the allowance (if necessary) */

1335:         /* We emit a Transfer event */

1440:         /* Get borrowBalance and borrowIndex */
```

#### [NC-03] Use battle-tested code

Use battle-tested code rather than reimplementing common patterns. Replace the `nonReentrant` modifier in `VToken.sol` with the `nonReentrant` modifier [from OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard-nonReentrant--), since it is well tested and optimized. The OpenZeppelin version of the modifier is already used in the Shortfall.sol `placeBid()` and `closeAuction()` functions.

```solidity
File: contracts/VToken.sol

33:     modifier nonReentrant() {
34:         require(_notEntered, "re-entered");
35:         _notEntered = false;
36:         _;
37:         _notEntered = true; // get a gas-refund post-Istanbul
38:     }
39:
```
