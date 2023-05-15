# Non-Critical Findings

## Naming Inconsistencies in loop index specifications
 
The variable names used to represent rewardDistributors.length within the `Comptroller.sol` contract are inconsistent, decreasing the searchability of code. 

These should be changed to the most commonly used name `rewardDistributorsCount` to maintain syntax consistency and increase readability. 

[Comptroller.sol#L93 - "RewardsDistributorsLength"](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L930)

[Comptroller.sol#L940 - "RewardsDistributorsLen"](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L940)

[Comptroller.sol#L1120 - "RewardsDistributorsLength"](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1120)

 
## Unused Custom Errors 
Within `ErrorReporter.sol` there are 13 instances of unused errors. Unnecessary or obsolete code should be removed or at least explained in the specification or in-line comments. Removal of these instances would improve readability. 

[ErrorReporter.sol#L7 - TransferComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L7)

[ErrorReporter.sol#L9 - TransferNotEnough()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L9) 

[ErrorReporter.sol#L10 - TransferTooMuch()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L10)

[ErrorReporter.sol#L12 - MintComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L12) 

[ErrorReporter.sol#L15 - RedeemComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L15 ) 

[ErrorReporter.sol#L19 - BorrowComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L19 )

[ErrorReporter.sol#L23 - RepayBorrowComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L23 )

[ErrorReporter.sol#L29 - LiquidateComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L29) 

[ErrorReporter.sol#L32 - LiquidateAccrueBorrowInterestFailed()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L32 )

[ErrorReporter.sol#L37 - LiquidateRepayBorrowFreshFailed()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L37)

[ErrorReporter.sol#L39 - LiquidateSeizeComptrollerRejection()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L39)

[ErrorReporter.sol#L42 - SetComptrollerOwnerCheck()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L42 )

[ErrorReporter.sol#L51 - ReduceReservesAdminCheck()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L51 )

# Low Findings

## Insufficient Loop Bounding in setActionPaused() could result in accidental DOS 
The loop bounding check `_ensureMaxLoops()` is insufficiently called within the `setActionsPaused()` function of `Comptroller.sol`. 

The check passes in `marketsCount` to bound a loop, but fails to implement proper bound checks on a subsequent nested loop. 
[Comptroller.sol#L895 - setActionPaused()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L895)


This can be resolved by passing `marketsCount * actionsCount` into the `_ensureMaxLoops()` call. 

 

## Incorrect initialization or update of maxLoopsLimit is irrecoverable 
Incorrectly initializing or updating `maxLoopsLimit` through `_setMaxLoopsLimit()` could result in an irrecoverably high threshold for loop iteration capacity. This is due to the `require` statement specifying that `maxLoopsLimit` may never be decreased. 

If this threshold is improperly set it would permanently undermine all `_ensureMaxLoops()` checks throughout the system, which could present unexpected behavior and denial of service vulnerabilities with no remediation. 

[MaxLoopsLimitHelper.sol#L26](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/MaxLoopsLimitHelper.sol#L26) 

It is recommended to introduce some guarded functionality that allows admins to adjust the iteration threshold in both directions, to protect against accidental misconfiguration. 


## Insufficient Loop Bounding in liquidateAccount() could result in accidental DOS 

The loop executed inside the `liquidateAccount()` function initiates the following control flow, iterating to the length of ordersCount: 
`Comptroller.liquidateAccount()` **-->** `VToken.forceLiquidateBorrow()`
 **-->** `VToken._liquidateBorrowFresh()` **-->** `Comptroller.preLiquidateHook()` **-->** `Comptroller._getCurrentLiquiditySnapshot()` **-->** `Comptroller._getHypotheticalLiquiditySnapshot()`

 **(Initiated from loop)**
`Comptroller.liquidateAccount()` **-->** `VToken.forceLiquidateBorrow()`
[Comptroller.sol#L669 - liquidateAccount()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L669)

 
`VToken.forceLiquidateBorrow()` **-->** `VToken._liquidateBorrowFresh()`
[VToken.sol#L459 - ForceLiquidateBorrow()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L459)

`VToken._liquidateBorrowFresh()` **-->** `Comptroller.preLiquidateHook()`
[VToken.sol#L1026 - _liquidateBorrowFresh()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1026)

`Comptroller.preLiquidateHook()` **-->** `Comptroller._getCurrentLiquiditySnapshot()`
[Comptroller.sol#L457 - PreLiquidateHook()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L457)

`Comptroller._getCurrentLiquiditySnapshot()` **-->** `Comptroller._getHypotheticalLiquiditySnapshot()`
[Comptroller.sol#L1281  - _getCurrentLiquiditySnapshot()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1281 )

**Nested / Unchecked Loop**
[Comptroller.sol#L1307 - _getHypotheticalLiquiditySnapshot()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1307 )


Inside the final function `_getHypotheticalLiquiditySnapshot()` another loop is executed (unchecked), iterating to the length of `assetCount`. 

 

The initial loop inside `liquidateAccount()` receives bound checks on `ordersCount` from the `_ensureMaxLoops()` call, although the nested nature of the second loop (inside `_getHypotheticalLiquiditySnapshot()`) renders the current bounding of loop iterations redundant, potentially resulting in denial of service if the resulting combine iterations are sufficiently high.  

This is because the nested loop multiples the iterations by a factor of the first max iterations (`ordersCount * assetCount`). 

To appropriately limit the iterations, `liquidateAccount()` should call `_ensureMaxLoops()` passing in `ordersCount * assetCount`. This would not affect alternate control flows, since calling `liquidateAccount()` always results in `_getHypotheticalLiquiditySnapshot()` being called. 