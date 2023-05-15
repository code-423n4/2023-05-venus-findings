## 1. VARIABLES CAN BE DECLARED OUTSIDE `for` LOOP TO SAVE GAS

In `Comptroller._getHypotheticalLiquiditySnapshot()` function, the variable (eg: uint256) can be declared outside the `for` loop to save gas.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1311

## 2. THE REVERT STRING IN THE `require` STATEMENT IS TOO LONG

In `RewardsDistributor.onlyComptroller` modifier, the description in the `require` statement is too long.

        require(address(comptroller) == msg.sender, "Only comptroller can call this function"); 
    
The descriptioin longer than 32 bytes utilizes another memory word which consumes more gas.
	
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L99

There is one more instance of this issue:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1072

## 3. REPETITIVE CODE CAN BE INCLUDED INTO AN `internal` FUNCTION TO REDUCE DEPLOYMENT GAS COST

In `Comptroller.sol` contract the following code snippet is being repeated multiple times.

        uint256 rewardDistributorsCount = rewardsDistributors.length;

This same code snippet is present in `preMintHook`, `preRedeemHook`, `preBorrowHook`, `preRepayHook` functions and few other functions in the `Comptroller.sol` contract. 

This code snippet can be included into a seperate internal function which will reduce the deployment gas cost.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L272
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L302

## 4. REDUNDANT CODE IS INCLUDED INSIDE THE `for` LOOP

In the `Comptroller.preRepayHook()` function, following code snippet is included inside the `for` loop.

    Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });

But it is not part of the iteration process of the `for` loop. 
Hence it can be placed outside the `for` loop similar to how it is done with the `Comptroller.preBorrowHook()` function.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L403

## 5. `calldata` PARAMETER CAN BE USED INPLACE OF THE `state` VARIABLE TO SAVE GAS

In the `Comptroller.setCloseFactor()` function, the event should use the calldata variable `newCloseFactorMantissa` instead of the state variable, which will save `SLOAD`.

        emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);

The above line should be corrected as below:

        emit NewCloseFactor(oldCloseFactorMantissa, newCloseFactorMantissa);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L709

## 6. `marketsCount` VARIABLE CAN BE INCREMENTED BY ONE INSTEAD OF CALLING THE `allMarkets` STATE VARIABLE ARRAY, TO RETRIEVE THE LENGTH

In the `Comptroller._addMarket()` function, once the new market is added, the marketsCount is assigned the value `allMarkets.length`. 
This requires `SLOAD` for the `allMarkets` state variable to be loaded. 
Instead the `marketsCount` can be incremented by one, since only one market is added during call to `_addMarket()` function.

        marketsCount = allMarkets.length;

Hence the above line of code can be changed as follows:

        marketsCount++;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1214

## 7. `repayAmount` SHOULD BE CALCULATED INSIDE THE `for` ONLY IF THE `borrowBalance != 0`

In the `Comptroller.healAccount()` function, the `repaymentAmount` by the liquidator is calculated as below:

    uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);

And the repayment happens as below:

            if (borrowBalance != 0) {
                market.healBorrow(liquidator, user, repaymentAmount);
            }

There is no need to calculate the `repayAmount` if the `borrowBalance == 0` inside the `for` loop for all the `userAssets`. 
Hence it is recommended to use the `repaymentAmount` calculation inside the `if` statement block as follows:

            if (borrowBalance != 0) {
                uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);
                market.healBorrow(liquidator, user, repaymentAmount);
            }

This will save runtime gas since no need to unnecasrily calculate the `repaymentAmount` if the `borrowBalance == 0`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L615
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L622-L624

## 8. `accountBorrowsPrev - badDebtDelta` ARITHMETIC OPERATION CAN BE REPLACED WITH `actualRepayAmount` VARIABLE

In the `VToken.healBorrow()` function, the following event is emitted

    emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);

Here the `accountBorrowsPrev - badDebtDelta == actualRepayAmount`. 
Hence there is no need for an additional arithmetic operation where as `actualRepayAmount` can be directly used to save gas.

The above `emit` can be udpated as follows to save gas:

    emit RepayBorrow(address(this), borrower, badDebtDelta, actualRepayAmount, totalBorrowsNew);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L420

## 9. REDUNDANT CHECK FOR `if (err != 0)`, SINCE `vToken.getAccountSnapshot(user)` ALWAYS RETURNS `NO_ERROR` WHICH TRANSLATES TO `err = 0`

In the `Comptroller._safeGetAccountSnapshot()` function, there is an redundant check for `if (err != 0) `.
But the `vToken.getAccountSnapshot(user)` always returns `err = 0`. Hence the below check is redundant and can be removed. This will save on the deployment gas cost and runtime gas cost.

        if (err != 0) {
            revert SnapshotError(address(vToken), user);
        }

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1413-L1415

There are two more instances of this issue:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L999-L1002
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1072