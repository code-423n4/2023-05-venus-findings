## 1. IN THE `Comptroller.exitMarket()` FUNCTION, THE `vTokenAddress` VARIABLE CAN BE DIRECTLY USED INPLACE OF `address(vToken)`. 

calling the `address(vToken)` is not required since the function calldata input parameter `vTokenAddress` can be directly used in the mapping.

        Market storage marketToExit = markets[address(vToken)];
		
Above code line can be updated as follows:

        Market storage marketToExit = markets[vTokenAddress];

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L201

## 2. `Typos` IN THE COMMENTS SHOULD BE CORRECTED

In the `Comptroller.preRepayHook()` function comments, the following @param comment should be corrected

    @param borrower The account which `would borrowed` the asset.

    `would` should be replace with `had`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L385

In the `Comptroller.preSeizeHook()` function, the below comment asks to check the if the borrowed token is also listed. But the function execution does not need to check the listing of the borrowed token. Hence the highlighted part (`borrowed token`) can be removed.

    * @custom:error MarketNotListed error is thrown if either collateral or `borrowed token` is not listed

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L482

## 3. NO INPUT VALIDATION FOR THE ZERO VALUE

In the `Comptroller.setCloseFactor()` function, there is no input validation for the `newCloseFactorMantissa`. There should be an input validation to make sure `newCloseFactorMantissa != 0`. Because if it is set to zero then entire liqudation process will be broken since `repayAmount > 0` can be bypassed with `repayAmount = 0`. Hence an attacker can `liquidate` the borrower without any repayment to the borrowed amount.

Similarly in the `Comptroller.setLiquidationIncentive()` function, there should be input validation for the `newLiquidationIncentiveMantissa` as well.

Similarly in the `Comptroller.setMinLiquidatableCollateral()` function, there should be input validation for the `newMinLiquidatableCollateral`. Since the default value is `0`, and by mistake if set to zero it will break the entire liquidation process, since the collateral amount needs to be greater than the `newMinLiquidatableCollateral` amount for liquidaiton to succeed, hence this condition will `always be true`.

Similarly in the `Comptroller.setCollateralFactor()` function, there should be input validation for `zero` value for the  `newCollateralFactorMantissa`. Since the default value is `0`, and by mistake if set to zero it will break the entire borrow process. Hence it is recommended to check and proceed only if  `newCollateralFactorMantissa != 0`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L708
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L788
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L916

## 4. VARIABLES NEED NOT BE INITIALIZED TO ZERO VALUE

The default value for variables is zero, so initializing them to zero is redundant operation.

        newMarket.collateralFactorMantissa = 0;
        newMarket.liquidationThresholdMantissa = 0;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L812-L813

## 5. CONSTANTS SHOULD BE DECLARED IN `UPPERCASE` LETTERS FOR EASE OF READABILITY AND UNDERSTANDING

In the `RewardsDistributor.sol` contract the `constant` variable is declared as follows:

    uint224 public constant rewardTokenInitialIndex = 1e36;

Above constant should be declared in uppercase letters as follows for ease of readability and understanding.

	uint224 public constant REWARDTOKENINITIALINDEX = 1e36;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L29

There is one more instance of this issue.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L20-L23

## 6. DIVISION BY ZERO POSSIBLE

In the `Comptroller.healAccount()` function, the `scaledBorrows.mantissa` could equal to `0`, if the `liquidationIncentiveMantissa` value is not set by the account manager and is left with the default value of `0`. 
In that case the following calculation to derive the percentage would revert without any error message due to division by zero.

    Exp memory percentage = div_(collateral, scaledBorrows);

Hence it is recommended to check the `scaledBorrows.mantissa` for `zero value` before performing the above calculation to derive `percentage`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L606

## 7. DIVISION BEFORE MULTIPLICATION CAN INTRODUCE ROUNDING ERROR

In the `Comptroller.liquidateCalculateSeizeTokens()` function, number of collateral asset tokens to sieze is calculated as follows:

        ratio = div_(numerator, denominator);

        seizeTokens = mul_ScalarTruncate(ratio, actualRepayAmount);

As it is seen above the division happens before the multiplication which could cause rounding error to the final resutl of the `seizeTokens`. 
Hence the returned `seizeTokens` amount could be less due to higher rounding error, in the event `actualRepayAmount` is large.

Hence the above calculation can be coded, such that multiplication happens prior to the division to mitigate any rounding error that could occur.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1107-L1109

## 8. LIQUIDATION THRESHOLD CAN BE CONFIGURED EQUAL TO COLLATERAL FACTOR, THUS ENABLING IMMEDIATE LIQUIDATION IN THE EVENT MAXIMUM AMOUNT IS BORROWED AGAINST THE COLLATERAL.

In `Comptroller.setCollateralFactor()` function, the liquidation threshold can be set equal to the Collateral Factor. 
In such scenario the maximum borrowed amount against the collateral can be immediately liquidated. 

        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
            revert InvalidLiquidationThreshold();
        }

It is recommended to configure the liquidation threshold to be more than the collateral factor to keep a buffer to save the users against market volatility. Hence above code snippet can be updated as follows:

        if (newLiquidationThresholdMantissa <= newCollateralFactorMantissa) {
            revert InvalidLiquidationThreshold();
        }

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L750-L752

## 9. `vToken` ADDRESS, `Market` IS NOT CHECKED FOR `address(0)` in the `Comptroller._checkActionPauseState()` FUNCTION.

In the `Comptroller._checkActionPauseState()` function, the `actionPaused(market, action)` will return `false` even if the market = address(0). 
Becuase `_actionPaused[market][action]` will return its default value of `false` for the non existing entry of the mapping. 
Because `actionPaused(market, action)` will return `false` in such scenario, it will not call the `revert` statement.

Hence the function execution will continue even if the `market = address(0)`. So for the function which use `_checkActionPauseState()` will continue execution even if the vToken market is address(0).
And it will only revert once a function is called on the `address(0)` market instance. Hence the gas will be wasted for the execution of operations prior to that.

It is recommended to check for the `address(0)` for the `market` instance and revert accordingly, inside the `Comptroller._checkActionPauseState()` function to mitigate this issue.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1431
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1154-L1156