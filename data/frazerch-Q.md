# [NC-01] Unused custom errors declared

Inside the contract `TokenErrorReporter`, there are 13 instances of custom errors being declared but not used inside the scope.

```
File: contracts/ErrorReporter.sol 

7:	error TransferComptrollerRejection(uint256 errorCode);
9:	error TransferNotEnough();
10:	error TransferTooMuch();
12:	error MintComptrollerRejection(uint256 errorCode);
15:	error RedeemComptrollerRejection(uint256 errorCode);
19:	error BorrowComptrollerRejection(uint256 errorCode);
23:	error RepayBorrowComptrollerRejection(uint256 errorCode);
29:	error LiquidateComptrollerRejection(uint256 errorCode);
32:	error LiquidateAccrueBorrowInterestFailed(uint256 errorCode);
37:	error LiquidateRepayBorrowFreshFailed(uint256 errorCode);
39:	error LiquidateSeizeComptrollerRejection(uint256 errorCode);
42:	error SetComptrollerOwnerCheck();
51	error ReduceReservesAdminCheck();
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L7 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L9 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L10  
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L12 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L15 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L19
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L23
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L29
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L32
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L37
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L39
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L42
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L51






# [NC-02] Inconsistent naming conventions throughout the code

Throughout the `Comptroller` contract, there are inconsistent naming conventions when assigning `rewardDistributors.length` to a local variable inside the function.

```
File: Comptroller.sol 
rewardsDistributorsLength:
930:    uint256 rewardsDistributorsLength = rewardsDistributors.length;
1120:   uint256 rewardsDistributorsLength = rewardsDistributors.length;

rewardsDistributorsLen:
940:    uint256 rewardsDistributorsLen = rewardsDistributors.length;

rewardDistributorsCount:
272:    uint256 rewardDistributorsCount = rewardsDistributors.length;
302:    uint256 rewardDistributorsCount = rewardsDistributors.length;
374:    uint256 rewardDistributorsCount = rewardsDistributors.length;
400:    uint256 rewardDistributorsCount = rewardsDistributors.length;
519:    uint256 rewardDistributorsCount = rewardsDistributors.length;
556:    uint256 rewardDistributorsCount = rewardsDistributors.length;
817:    uint256 rewardDistributorsCount = rewardsDistributors.length;
```
`rewardsDistributorsLength`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L930 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1120
 
`rewardsDistributorsLen`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L940 

`rewardDistributorsCount`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L272 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L302 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L374  
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L400 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L519 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L556 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L817 


# [L-01] `_ensureMaxLoops` does not account for nested loops inside `setActionPaused`

Inside the `setActionPaused` function, there is a `_ensureMaxLoops` check which ensures that the outer `for` loop bound of `marketsCount` does not exceed the max loop limit. Within the loop, there is a second `for` loop, which iterates through an array of `Action` enums, which is not accounted for inside the `_ensureMaxLoops` check.

This means that the function has the potential to perform `marketList.length * actionsList.length` amount of loops, which can be outside the bounds of the `maxLoopLimit`. This has the potential to cause out-of-gas errors which may lead to a denial of service.

Ensure that the `_ensureMaxLoops` check accounts for the iteration values of both loops.

```
File: Comptroller.sol
895:    	_ensureMaxLoops(marketsCount);
897:    	for (uint256 marketIdx; marketIdx < marketsCount; ++marketIdx) {
898:    	for (uint256 actionIdx; actionIdx < actionsCount; ++actionIdx) {
899:   	_setActionPaused(address(marketsList[marketIdx]), actionsList[actionIdx], paused);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L895-L899 


# [L-02] `_ensureMaxLoops` does not account for nested loops inside `liquidateAccount`

Inside the `liquidateAccount` function, there is a `_ensureMaxLoops` check which ensures that the outer `for` loop bound of `ordersCount` does not exceed the max loop limit. Within the control flow inside the loop, there is a second `for` loop, which iterates through an array of assets that the provided account has, which is not accounted for inside the `_ensureMaxLoops` check.

This means that the function has the potential to perform `orders.length * assets.length` amount of loops, which can be outside the bounds of the `maxLoopLimit`. This has the potential to cause out-of-gas errors, which may lead to a denial of service. 

Ensure that the `_ensureMaxLoops` check accounts for the iteration values of both loops.

```
File: Comptroller.sol
641: 	function liquidateAccount(address borrower, LiquidationOrder[] calldata orders) external {
667:	_ensureMaxLoops(ordersCount);
669:   	for (uint256 i; i < ordersCount; ++i)

1296:	function _getHypotheticalLiquiditySnapshot(
1305:  uint256 assetsCount = assets.length;
1307: 	for (uint256 i; i < assetsCount; ++i) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L667-L669 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1305-L1307 




# [L-03] No functionality to decrease `maxLoopsLimit`

The contract `MaxLoopsLimitHelper` does not contain any functionality to decrease the maxLoopLimit once it has been set by the function `_setMaxLoopsLimit`. This is due to a `require` statement which checks that the new value of `limit` that is input to the function must be greater than the previous `maxLoopsLimit`. 

If a value is accidentally set which is too high, this cannot be decreased and may result in every `_ensureMaxLoops` check that is performed becoming redundant, which may result in multiple entry points for which a denial of service could occur. 

A mitigation to this could be to add a specific function which will allow authorised users to reduce the `maxLoopLimit`.

```
File:  MaxLoopsLimitHelper.sol
25:  	function _setMaxLoopsLimit(uint256 limit) internal {
26: 	require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L26 
