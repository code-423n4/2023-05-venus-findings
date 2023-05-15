# Low Risk

|#|Title|
|---|---|
|L-1|Missing validation|
|L-2|Simple checks should be before complex one|
|L-3|Unnecessary gas burning|

|L-|Validation should be at the top of the function|

## L-1 Missing validation

In code below `address cmptroller` validation may be added as in `ReserveHelpers.sol`
`require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");`
Additionally, the `uint256 amount` validation `require(amount > 0, "ProtocolShareReserve: Invalida amount");`, without which the calculations will be performed, which will eventually return an error, burning only gas.
```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol
66:     function releaseFunds(
67:         address comptroller, //audit:low
68:         address asset,
69:         uint256 amount //audit:low
70:     ) external returns (uint256) {
71:         require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
72:         require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
73: 
74:         assetsReserves[asset] -= amount;
75:         poolsAssetsReserves[comptroller][asset] -= amount;
76:         uint256 protocolIncomeAmount = mul_(
77:             Exp({ mantissa: amount }),
78:             div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
79:         ).mantissa;
80: 
81:         IERC20Upgradeable(asset).safeTransfer(protocolIncome, protocolIncomeAmount);
82:         IERC20Upgradeable(asset).safeTransfer(riskFund, amount - protocolIncomeAmount);
83: 
84:         // Update the pool asset's state in the risk fund for the above transfer.
85:         IRiskFund(riskFund).updateAssetsState(comptroller, asset);
86: 
87:         emit FundsReleased(comptroller, asset, amount);
88: 
89:         return amount;
90:     }
```

## L-2 Simple checks should be before complex one
```solidity
File: contracts/RiskFund/ReserveHelpers.sol
38:     function getPoolAssetReserve(address comptroller, address asset) external view returns (uint256) {
39:         require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
40:         require(asset != address(0), "ReserveHelpers: Asset address invalid"); //audit:low
41:         return poolsAssetsReserves[comptroller][asset];
42:     }
```
```diff
@@ -1,34 +1,34 @@
 File: contracts/VToken.sol
-1025:         /* Fail if liquidate not allowed */
-1026:         comptroller.preLiquidateHook(
-1027:             address(this),
-1028:             address(vTokenCollateral),
-1029:             borrower,
-1030:             repayAmount,
-1031:             skipLiquidityCheck
-1032:         );
-1033: 
-1034:         /* Verify market's block number equals current block number */
-1035:         if (accrualBlockNumber != _getBlockNumber()) {
-1036:             revert LiquidateFreshnessCheck();
-1037:         }
-1038: 
-1039:         /* Verify vTokenCollateral market's block number equals current block number */
-1040:         if (vTokenCollateral.accrualBlockNumber() != _getBlockNumber()) {
-1041:             revert LiquidateCollateralFreshnessCheck();
-1042:         }
-1043:         //audit:non simple checks should be before complex one
-1044:         /* Fail if borrower = liquidator */
-1045:         if (borrower == liquidator) {
-1046:             revert LiquidateLiquidatorIsBorrower();
-1047:         }
-1048:         //audit:non simple checks should be before complex one
-1049:         /* Fail if repayAmount = 0 */
-1050:         if (repayAmount == 0) {
-1051:             revert LiquidateCloseAmountIsZero();
-1052:         }
-1053:         //audit:non simple checks should be before complex one
-1054:         /* Fail if repayAmount = -1 */
-1055:         if (repayAmount == type(uint256).max) {
-1056:             revert LiquidateCloseAmountIsUintMax();
-1057:         }
+1025:         //audit:non simple checks should be before complex one
+1026:         /* Fail if borrower = liquidator */
+1027:         if (borrower == liquidator) {
+1028:             revert LiquidateLiquidatorIsBorrower();
+1029:         }
+1030:         //audit:non simple checks should be before complex one
+1031:         /* Fail if repayAmount = 0 */
+1032:         if (repayAmount == 0) {
+1033:             revert LiquidateCloseAmountIsZero();
+1034:         }
+1035:         //audit:non simple checks should be before complex one
+1036:         /* Fail if repayAmount = -1 */
+1037:         if (repayAmount == type(uint256).max) {
+1038:             revert LiquidateCloseAmountIsUintMax();
+1039:         }
+1040:         /* Verify market's block number equals current block number */
+1041:         if (accrualBlockNumber != _getBlockNumber()) {
+1042:             revert LiquidateFreshnessCheck();
+1043:         }
+1044: 
+1045:         /* Verify vTokenCollateral market's block number equals current block number */
+1046:         if (vTokenCollateral.accrualBlockNumber() != _getBlockNumber()) {
+1047:             revert LiquidateCollateralFreshnessCheck();
+1048:         }
+1049: 
+1050:         /* Fail if liquidate not allowed */
+1051:         comptroller.preLiquidateHook(
+1052:             address(this),
+1053:             address(vTokenCollateral),
+1054:             borrower,
+1055:             repayAmount,
+1056:             skipLiquidityCheck
+1057:         );
```

## L-3 Not checking address(0) causes unnecessary gas burning

Without checking `RewardsDistributor` for `address(0)` the function will pass without error adding `address(0)` as `rewordsDistributor`. This will add a broken contract to the `rewardsDistributors` and unnecessary checking in each loop.

```solidity
File: contracts/Comptroller.sol
927:     function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner {
928:         require(!rewardsDistributorExists[address(_rewardsDistributor)], "already exists");
929: 
930:         uint256 rewardsDistributorsLength = rewardsDistributors.length;
931: 
932:         for (uint256 i; i < rewardsDistributorsLength; ++i) {
933:             address rewardToken = address(rewardsDistributors[i].rewardToken());
934:             require(
935:                 rewardToken != address(_rewardsDistributor.rewardToken()),
936:                 "distributor already exists with this reward"
937:             );
938:         }
939: 
940:         uint256 rewardsDistributorsLen = rewardsDistributors.length;
941:         _ensureMaxLoops(rewardsDistributorsLen + 1);
942: 
943:         rewardsDistributors.push(_rewardsDistributor);
944:         rewardsDistributorExists[address(_rewardsDistributor)] = true;
945: 
946:         uint256 marketsCount = allMarkets.length;
947: 
948:         for (uint256 i; i < marketsCount; ++i) {
949:             _rewardsDistributor.initializeMarket(address(allMarkets[i]));
950:         }
951: 
952:         emit NewRewardsDistributor(address(_rewardsDistributor));
953:     }
```

# Non-critical Risk

|#|Title|
|---|---|
|N-1|Redundant function|
|N-2|Redundant return statement|
|N-3|Consider adding indexing|
|N-4|Consider marking as abstract|
|N-5|No consistency in the naming of constants|
|N-6|`preRepayHook` refactoring|
|N-7|Consider refactoring duplicate code|
|N-8|Consider `isMarketListed` refactoring|
|N-9|Redundant local variable|

## N-1 Redundant function `function setPoolRegistry(address _poolRegistry) external`
The function occurs in two contracts `ProtocolShareReserve.sol` and `RiskFund.sol`, both inherit from `ReserveHelpers.sol` which means that it can be safely moved to the parent contract. In addition, the event `PoolRegistryUpdated` triggered by it is also duplicated and can be moved.

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol
25:     event PoolRegistryUpdated(address indexed oldPoolRegistry, address indexed newPoolRegistry);
...
File: contracts/RiskFund/ProtocolShareReserve.sol
53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {
54:         require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");
55:         address oldPoolRegistry = poolRegistry;
56:         poolRegistry = _poolRegistry;
57:         emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
58:     }
```
```solidity
File: contracts/RiskFund/RiskFund.sol
38:     event PoolRegistryUpdated(address indexed oldPoolRegistry, address indexed newPoolRegistry);
...
File: contracts/RiskFund/RiskFund.sol
099:     function setPoolRegistry(address _poolRegistry) external onlyOwner {
100:         require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
101:         address oldPoolRegistry = poolRegistry;
102:         poolRegistry = _poolRegistry;
103:         emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
104:     }
```

## N-2 Redundant return statement
```solidity
File: contracts/Pool/PoolRegistry.sol
213:     function createRegistryPool(
214:         string calldata name,
215:         address beaconAddress,
216:         uint256 closeFactor,
217:         uint256 liquidationIncentive,
218:         uint256 minLiquidatableCollateral,
219:         address priceOracle,
220:         uint256 maxLoopsLimit,
221:         address accessControlManager
222:     ) external virtual returns (uint256 index, address proxyAddress) {
...
248:         return (poolId, proxyAddress);
249:     }
```

## N-3 Consider adding indexing to address fields    
```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol
22:     event FundsReleased(address comptroller, address asset, uint256 amount);
```
```solidity
File: contracts/BaseJumpRateModelV2.sol
55:     error Unauthorized(address sender, address calledContract, string methodSignature);
```

## N-4 Consider marking as abstract
`ReserveHelpers.sol` can be abstract, as it only serves as a base class for `RiskFund.sol` and `ProtocolShareReserve.sol`.

## N-5 No consistency in the naming of constants
It is accepted practice to name variables in upper case. In the project, the naming is different in different places, which may have a bad effect on the readability of the code.
```solidity
File: contracts/BaseJumpRateModelV2.sol
13:     uint256 private constant BASE = 1e18;
...
23:     uint256 public constant blocksPerYear = 10512000; //audit:low naming
```

## N-6 `preRepayHook` refactoring

Line `Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });` can be moved outside the loop, because its value does not depend on it. Similar to the `preBorrowHook` function.

```diff
 File: contracts/Comptroller.sol
 400:         uint256 rewardDistributorsCount = rewardsDistributors.length;
-401: 
+401:         Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
 402:         for (uint256 i; i < rewardDistributorsCount; ++i) {
-403:             Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
-404:             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
-405:             rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
-406:         }
-407:     }
+403:             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
+404:             rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
+405:         }
```

## N-7 Consider refactoring duplicate code

Duplication in functions `preMintHook` and `preRedeemHook`.

```solidity
File: contracts/Comptroller.sol
271:         // Keep the flywheel moving //audit:non refactor, same block of code in preRedeemHook
272:         uint256 rewardDistributorsCount = rewardsDistributors.length;
273: 
274:         for (uint256 i; i < rewardDistributorsCount; ++i) {
275:             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
276:             rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
277:         }
```
```solidity
File: contracts/Comptroller.sol
301:         // Keep the flywheel moving //audit:non refactor, same block of code in preMintHook
302:         uint256 rewardDistributorsCount = rewardsDistributors.length;
303: 
304:         for (uint256 i; i < rewardDistributorsCount; ++i) {
305:             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
306:             rewardsDistributors[i].distributeSupplierRewardToken(vToken, redeemer);
307:         }
```

Duplication in functions `preBorrowHook` and `preRepayHook`.

The second block of code contains `Exp memory borrowIndex`, which, as mentioned in N-6, can be moved outside the loop, and then similar blocks can be moved to an internal function.

```solidity
File: contracts/Comptroller.sol
373:         // Keep the flywheel moving
374:         uint256 rewardDistributorsCount = rewardsDistributors.length;
375: 
376:         for (uint256 i; i < rewardDistributorsCount; ++i) {
377:             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
378:             rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
379:         }
```
```solidity
File: contracts/Comptroller.sol
399:         // Keep the flywheel moving
400:         uint256 rewardDistributorsCount = rewardsDistributors.length;
401: 
402:         for (uint256 i; i < rewardDistributorsCount; ++i) {
403:             Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() }); //audit:non can be moved
404:             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
405:             rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
406:         }
```

Duplication in functions `preSeizeHook` and `preTransferHook`.

```solidity
File: contracts/Comptroller.sol
518:         // Keep the flywheel moving
519:         uint256 rewardDistributorsCount = rewardsDistributors.length;
520: 
521:         for (uint256 i; i < rewardDistributorsCount; ++i) {
522:             rewardsDistributors[i].updateRewardTokenSupplyIndex(vTokenCollateral);
523:             rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, borrower);
524:             rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, liquidator);
525:         }
...
File: contracts/Comptroller.sol
555:         // Keep the flywheel moving
556:         uint256 rewardDistributorsCount = rewardsDistributors.length;
557: 
558:         for (uint256 i; i < rewardDistributorsCount; ++i) {
559:             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
560:             rewardsDistributors[i].distributeSupplierRewardToken(vToken, src);
561:             rewardsDistributors[i].distributeSupplierRewardToken(vToken, dst);
562:         }
```

## N-8 Consider `isMarketListed` refactoring

Checking whether the market is listed occurs many times in the code in the form of a simple `if` and several times as the `isMarketListed` function. This can be reduced to one `isMarketListed` function.

```diff
@@ -1,4 +1,4 @@
 File: contracts/Comptroller.sol
-1047:     function isMarketListed(VToken vToken) external view returns (bool) {
-1048:         return markets[address(vToken)].isListed;
+1047:     function isMarketListed(address vToken) external view returns (bool) {
+1048:         return markets[vToken].isListed;
 1049:     }
 ```
`if` statement example:

```solidity
File: contracts/Comptroller.sol
497:         if (!markets[vTokenCollateral].isListed) {
498:             revert MarketNotListed(vTokenCollateral);
499:         }
```

The function in its original form occurs 3 times in the project, while `if` occurs 16 times.

## N-9 Redundant local variable
```diff
@@ -3,9 +3,9 @@ File: contracts/Comptroller.sol
 837:         _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");
 838: 
 839:         uint256 numMarkets = vTokens.length;
-840:         uint256 numBorrowCaps = newBorrowCaps.length; //audit:non used once
+840:         
 841: 
-842:         require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");
+842:         require(numMarkets != 0 && numMarkets == newBorrowCaps.length, "invalid input");
 843: 
 844:         _ensureMaxLoops(numMarkets);
 845: 
 ```
