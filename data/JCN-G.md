# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.13`, `optimizer on`, and `200 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions, with all the optimizations applied:
| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| ERC20.decreaseAllowance |  36685  |  36615  | 70 | 
| ERC20.increaseAllowance |  36636  |  36560  | 76 | 
| ERC20.transfer |  146501  |  145334  | 1167 | 
| ERC20.transferFrom |  92902  |  92435  | 467 | 
| Comptroller.addRewardsDistributor |  175094  |  174683  | 411 | 
| Comptroller.enterMarkets |  132444  |  132238  | 206 | 
| Comptroller.exitMarket |  69698  |  67959  | 1739 | 
| Comptroller.healAccount |  223131  |  218001  | 5130 | 
| Comptroller.liquidateAccount |  233098  |  223536  | 9562 | 
| Comptroller.preBorrowHook |  102587  |  100579  | 2008 | 
| Comptroller.supportMarket |  103153  |  98338  | 4815 | 
| PoolRegistry.addMarket |  1378668  |  1334768  | 43900 | 
| PoolRegistry.createRegistryPool |  664419  |  618956  | 45463 | 
| PoolRegistry.updatePoolMetadata |  115095  |  114348  | 747 | 
| ProtocolShareReserve.releaseFunds |  172312  |  171992  | 320 | 
| ProtocolShareReserve.updateAssetsState |  89962  |  89816  | 146 | 
| RewardsDistributor.claimRewardToken |  276621  |  276519  | 102 | 
| RewardsDistributor.setRewardTokenSpeeds |  208377  |  207399  | 978 | 
| RiskFund.swapPoolsAssets |  366040  |  357571  | 8469 | 
| Shortfall.closeAuction |  191937  |  188607  | 3330 | 
| Shortfall.placeBid |  185429  |  183262  | 2167 | 
| Shortfall.restartAuction |  114761  |  114056  | 705 | 
| Shortfall.startAuction |  285153  |  284579  | 574 | 
| VToken.borrow |  323336  |  319410  | 3926 | 
| VToken.liquidateBorrow |  293130  |  285753  | 7377 | 
| VToken.mint |  155902  |  153899  | 2003 | 
| VToken.redeem |  184107  |  182396  | 1711 | 
| VToken.reduceReserves |  184956  |  184375  | 581 | 
| VToken.repayBorrow |  137882  |  136740  | 1142 | 

**Total gas saved across all listed functions: 149292**

*Notes*: 

- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diffs for each contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/403d8b821361aac8d809c263b0784657).

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01](#state-variables-only-set-in-the-constructor-should-be-declared-immutable) | State variables only set in the constructor should be declared immutable | 1 | 
| [G-02](#state-variables-can-be-packed-to-use-fewer-storage-slots) | State variables can be packed to use fewer storage slots | 6 |
| [G-03](#structs-can-be-packed-to-use-fewer-storage-slots) | Structs can be packed to use fewer storage slots | 3 |
| [G-04](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 40 |
| [G-05](#cache-state-variables-outside-of-loop-to-avoid-reading-storage-on-every-iteration) | Cache state variables outside of loop to avoid reading storage on every iteration | 2 |
| [G-06](#avoid-emitting-storage-values) | Avoid emitting storage values | 9 |
| [G-07](#use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated) | Use calldata instead of memory for function arguments that do not get mutate | 3 |  
| [G-08](#refactor-internal-function-to-avoid-unnecessary-sload) | Refactor internal function to avoid unnecessary SLOAD | 3 |
| [G-09](#return-values-from-external-calls-can-be-cached-to-avoid-unnecessary-call) | Return values from external calls can be cached to avoid unnecessary call | 2 |
| [G-10](#a-mapping-is-more-efficient-than-an-array) | A mapping is more efficient than an array | 2 |
| [G-11](#move-storage-pointer-to-top-of-function-to-avoid-offset-calculation) | Move storage pointer to top of function to avoid offset calculation | 1 |
| [G-12](#move-calldata-pointer-to-top-of-for-loop-to-avoid-offset-calculations) | Move calldata pointer to top of for loop to avoid offset calculations | 1 |
| [G-13](#using-storage-instead-of-memory-for-structsarrays-saves-gas) | Using storage instead of memory for structs/arrays saves gas | 1 |
| [G-14](#multiple-accesses-of-a-mappingarray-should-use-a-storage-pointer) | Multiple accesses of a mapping/array should use a storage pointer | - |
| [G-15](#use-do-while-loops-instead-of-for-loops) | Use `do while` loops instead of for loops | - |
| [G-16](#use-assembly-to-perform-efficient-back-to-back-calls) | Use assembly to perform efficient back-to-back calls | 1 |

## State variables only set in the constructor should be declared immutable
The solidity compiler will directly embed the values of immutable variables into your contract bytecode and therefore will save you from incurring a `Gsset (20000 gas)` when you set storage variables in the constructor, a `Gcoldsload (2100 gas)` when you access storage variables for the first time in a transaction, and a `Gwarmaccess (100 gas)` for each subsequent access to that storage slot.

Total Instances: `1`

Estimated Gas Saved: `1 * 2100 = 2100`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L18

```solidity
File: contracts/BaseJumpRateModelV2.sol
18:    IAccessControlManagerV8 public accessControlManager;
```diff
diff --git a/contracts/BaseJumpRateModelV2.sol b/contracts/BaseJumpRateModelV2.sol
index 68b535a..bd83624 100644
--- a/contracts/BaseJumpRateModelV2.sol
+++ b/contracts/BaseJumpRateModelV2.sol
@@ -15,7 +15,7 @@ abstract contract BaseJumpRateModelV2 is InterestRateModel {
     /**
      * @notice The address of the AccessControlManager contract
      */
-    IAccessControlManagerV8 public accessControlManager;
+    IAccessControlManagerV8 public immutable accessControlManager;

     /**
      * @notice The approximate number of blocks per year that is assumed by the interest rate model
```

## State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Total Instances: `6`

Estimated Gas Saved: `6 (slots) * 2000 = 12000`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L28-L38

### Pack `multiplierPerBlock`, `baseRatePerBlock`, and `jumpMultiplierPerBlock` into one storage slot to save 2 SLOTs (~2000 gas)

**Note: we will need to reduce the uint type for `multiplierPerBlock`, `baseRatePerBlock`, and `jumpMultiplierPerBlock` for this optimization to work**.
```solidity
File: contracts/BaseJumpRateModelV2.sol
28:    uint256 public multiplierPerBlock;
29:
30:    /**
31:     * @notice The base interest rate which is the y-intercept when utilization rate is 0
32:     */
33:    uint256 public baseRatePerBlock;
34:
35:    /**
36:     * @notice The multiplierPerBlock after hitting a specified utilization point
37:     */
38:    uint256 public jumpMultiplierPerBlock;
```
```diff
diff --git a/contracts/BaseJumpRateModelV2.sol b/contracts/BaseJumpRateModelV2.sol
index 68b535a..3cefbf1 100644
--- a/contracts/BaseJumpRateModelV2.sol
+++ b/contracts/BaseJumpRateModelV2.sol
@@ -25,17 +25,17 @@ abstract contract BaseJumpRateModelV2 is InterestRateModel {
     /**
      * @notice The multiplier of utilization rate that gives the slope of the interest rate
      */
-    uint256 public multiplierPerBlock;
+    uint80 public multiplierPerBlock;

     /**
      * @notice The base interest rate which is the y-intercept when utilization rate is 0
      */
-    uint256 public baseRatePerBlock;
+    uint80 public baseRatePerBlock;

     /**
      * @notice The multiplierPerBlock after hitting a specified utilization point
      */
-    uint256 public jumpMultiplierPerBlock;
+    uint80 public jumpMultiplierPerBlock;

     /**
      * @notice The utilization point at which the jump multiplier is applied
@@ -154,9 +154,9 @@ abstract contract BaseJumpRateModelV2 is InterestRateModel {
         uint256 jumpMultiplierPerYear,
         uint256 kink_
     ) internal {
-        baseRatePerBlock = baseRatePerYear / blocksPerYear;
-        multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
-        jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
+        baseRatePerBlock = uint80(baseRatePerYear / blocksPerYear);
+        multiplierPerBlock = uint80((multiplierPerYear * BASE) / (blocksPerYear * kink_));
+        jumpMultiplierPerBlock = uint80(jumpMultiplierPerYear / blocksPerYear);
         kink = kink_;

         emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L30-L31

### Pack `minAmountToConvert` and `convertibleBaseAsset` into one storage slot to save 1 SLOT (~2000 gas)

**Note: we will need to reduce the uint type for `minAmountToConvert` for this optimization to work**.
```solidity
File: contracts/RiskFund/RiskFund.sol
30:    uint256 private minAmountToConvert;
31:    address private convertibleBaseAsset;
```
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..0c7bcd3 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -27,8 +27,8 @@ contract RiskFund is
     using SafeERC20Upgradeable for IERC20Upgradeable;

     address private pancakeSwapRouter;
-    uint256 private minAmountToConvert;
     address private convertibleBaseAsset;
+    uint96 private minAmountToConvert;
     address private shortfall;

     // Store base asset's reserve for specific pool
@@ -86,7 +86,7 @@ contract RiskFund is
         __AccessControlled_init_unchained(accessControlManager_);

         pancakeSwapRouter = pancakeSwapRouter_;
-        minAmountToConvert = minAmountToConvert_;
+        minAmountToConvert = uint96(minAmountToConvert_);
         convertibleBaseAsset = convertibleBaseAsset_;

         _setMaxLoopsLimit(loopsLimit_);
@@ -138,7 +138,7 @@ contract RiskFund is
         _checkAccessAllowed("setMinAmountToConvert(uint256)");
         require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
         uint256 oldMinAmountToConvert = minAmountToConvert;
-        minAmountToConvert = minAmountToConvert_;
+        minAmountToConvert = uint96(minAmountToConvert_);
         emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/ComptrollerStorage.sol#L59-L89

### Pack `oracle` & `closeFactorMantissa` into one slot and `liquidationIncentiveMantissa` & `minLiquidatableCollateral` into one slot to save 2 SLOTs (~4000 gas)

**Note: we will need to reduce the `uint` type for `closeFactorMantissa`, `liquidationIncentiveMantissa`, and `minLiquidatableCollateral` for this optimization to work**.
```solidity
File: contracts/ComptrollerStorage.sol
59:    PriceOracle public oracle;
60:
61:    /**
62:     * @notice Multiplier used to calculate the maximum repayAmount when liquidating a borrow
63:     */
64:    uint256 public closeFactorMantissa;
65:
66:    /**
67:     * @notice Multiplier representing the discount on collateral that a liquidator receives
68:     */
69:    uint256 public liquidationIncentiveMantissa;
...
88:    /// @notice Minimal collateral required for regular (non-batch) liquidations
89:    uint256 public minLiquidatableCollateral;
```
```diff
diff --git a/contracts/ComptrollerStorage.sol b/contracts/ComptrollerStorage.sol
index 31cca94..a73def5 100644
--- a/contracts/ComptrollerStorage.sol
+++ b/contracts/ComptrollerStorage.sol
@@ -61,12 +61,15 @@ contract ComptrollerStorage {
     /**
      * @notice Multiplier used to calculate the maximum repayAmount when liquidating a borrow
      */
-    uint256 public closeFactorMantissa;
+    uint96 public closeFactorMantissa;

     /**
      * @notice Multiplier representing the discount on collateral that a liquidator receives
      */
-    uint256 public liquidationIncentiveMantissa;
+    uint128 public liquidationIncentiveMantissa;
+
+    /// @notice Minimal collateral required for regular (non-batch) liquidations
+    uint128 public minLiquidatableCollateral;

     /**
      * @notice Per-account mapping of "assets you are in"
@@ -85,8 +88,6 @@ contract ComptrollerStorage {
     /// @notice Borrow caps enforced by borrowAllowed for each vToken address. Defaults to zero which restricts borrowing.
     mapping(address => uint256) public borrowCaps;

-    /// @notice Minimal collateral required for regular (non-batch) liquidations
-    uint256 public minLiquidatableCollateral;

     /// @notice Supply caps enforced by mintAllowed for each vToken address. Defaults to zero which corresponds to minting notAllowed
     mapping(address => uint256) public supplyCaps;
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..e697717 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -705,7 +705,7 @@ contract Comptroller is
         require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");

         uint256 oldCloseFactorMantissa = closeFactorMantissa;
-        closeFactorMantissa = newCloseFactorMantissa;
+        closeFactorMantissa = uint96(newCloseFactorMantissa);
         emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
     }

@@ -785,7 +785,7 @@ contract Comptroller is
         uint256 oldLiquidationIncentiveMantissa = liquidationIncentiveMantissa;

         // Set liquidation incentive to new incentive
-        liquidationIncentiveMantissa = newLiquidationIncentiveMantissa;
+        liquidationIncentiveMantissa = uint128(newLiquidationIncentiveMantissa);

         // Emit event with old incentive, new incentive
         emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);
@@ -913,7 +913,7 @@ contract Comptroller is
         _checkAccessAllowed("setMinLiquidatableCollateral(uint256)");

         uint256 oldMinLiquidatableCollateral = minLiquidatableCollateral;
-        minLiquidatableCollateral = newMinLiquidatableCollateral;
+        minLiquidatableCollateral = uint128(newMinLiquidatableCollateral);
         emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L61-L79C20

### Pack `comptroller` and `accrualBlockNumber` into 1 storage slot to save 1 SLOT (~2000 gas)

**Note: These two state variables are both read in numerous functions and would therefore benefit from being packed in storage. `uint96` should be big enough for a value that holds block numbers. However, since slight changes would need to be done in the tests for this optimization to work, it will not be included in the final diffs. It is mentioned here for completeness**.

*Functions that would benefit from packing `comptroller` and `accrualBlockNumber` into the same storage slot: [_mintFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L743), [_redeemFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L800), [_borrowFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L877), [_repayBorrowFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L930), [_liquidateBorrowFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1018), and [_reduceReservesFresh](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1200)*.
```solidity
File: contracts/VTokenInterfaces.sol
61:    ComptrollerInterface public comptroller;
...
76:    /**
77:     * @notice Block number that interest was last accrued at
78:     */
79:    uint256 public accrualBlockNumber;
```
```diff
diff --git a/contracts/VTokenInterfaces.sol b/contracts/VTokenInterfaces.sol
index 8f578d7..ad80c1a 100644
--- a/contracts/VTokenInterfaces.sol
+++ b/contracts/VTokenInterfaces.sol
@@ -60,6 +60,11 @@ contract VTokenStorage {
      */
     ComptrollerInterface public comptroller;

+    /**
+     * @notice Block number that interest was last accrued at
+     */
+    uint96 public accrualBlockNumber;
+
     /**
      * @notice Model which tells what the current interest rate should be
      */
@@ -73,10 +78,6 @@ contract VTokenStorage {
      */
     uint256 public reserveFactorMantissa;

-    /**
-     * @notice Block number that interest was last accrued at
-     */
-    uint256 public accrualBlockNumber;

     /**
      * @notice Accumulator of the total earned interest rate since the opening of the market
```

## Structs can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Total Instances: `3`

Estimated Gas Saved: `3 (slots) * 2000 = 6000`

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/ComptrollerStorage.sol#L29-L42

### Pack `isListed`, `collateralFactorMantissa`, and `liquidationThresholdMantissa` into one storage slot to save 2 SLOTs (~4000 gas)

**Note: we will need to reduce the `uint` type for `collateralFactorMantissa` and `liquidationThresholdMantissa` for this optimization to work**.
```solidity
File: ComptrollerStorage.sol
29:    struct Market {
30:        // Whether or not this market is listed
31:        bool isListed;
32:        //  Multiplier representing the most one can borrow against their collateral in this market.
33:        //  For instance, 0.9 to allow borrowing 90% of collateral value.
34:        //  Must be between 0 and 1, and stored as a mantissa.
35:        uint256 collateralFactorMantissa;
36:        //  Multiplier representing the collateralization after which the borrow is eligible
37:        //  for liquidation. For instance, 0.8 liquidate when the borrow is 80% of collateral
38:        //  value. Must be between 0 and collateral factor, stored as a mantissa.
39:        uint256 liquidationThresholdMantissa;
40:        // Per-market mapping of "accounts in this asset"
41:        mapping(address => bool) accountMembership;
42:    }
```
```diff
diff --git a/contracts/ComptrollerStorage.sol b/contracts/ComptrollerStorage.sol
index 31cca94..6ab4f87 100644
--- a/contracts/ComptrollerStorage.sol
+++ b/contracts/ComptrollerStorage.sol
@@ -32,11 +32,11 @@ contract ComptrollerStorage {
         //  Multiplier representing the most one can borrow against their collateral in this market.
         //  For instance, 0.9 to allow borrowing 90% of collateral value.
         //  Must be between 0 and 1, and stored as a mantissa.
-        uint256 collateralFactorMantissa;
+        uint128 collateralFactorMantissa;
         //  Multiplier representing the collateralization after which the borrow is eligible
         //  for liquidation. For instance, 0.8 liquidate when the borrow is 80% of collateral
         //  value. Must be between 0 and collateral factor, stored as a mantissa.
-        uint256 liquidationThresholdMantissa;
+        uint120 liquidationThresholdMantissa;
         // Per-market mapping of "accounts in this asset"
         mapping(address => bool) accountMembership;
     }
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..2dd0d92 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -758,13 +758,13 @@ contract Comptroller is

         uint256 oldCollateralFactorMantissa = market.collateralFactorMantissa;
         if (newCollateralFactorMantissa != oldCollateralFactorMantissa) {
-            market.collateralFactorMantissa = newCollateralFactorMantissa;
+            market.collateralFactorMantissa = uint128(newCollateralFactorMantissa);
             emit NewCollateralFactor(vToken, oldCollateralFactorMantissa, newCollateralFactorMantissa);
         }

         uint256 oldLiquidationThresholdMantissa = market.liquidationThresholdMantissa;
         if (newLiquidationThresholdMantissa != oldLiquidationThresholdMantissa) {
-            market.liquidationThresholdMantissa = newLiquidationThresholdMantissa;
+            market.liquidationThresholdMantissa = uint120(newLiquidationThresholdMantissa);
             emit NewLiquidationThreshold(vToken, oldLiquidationThresholdMantissa, newLiquidationThresholdMantissa);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L17-L20

### Pack `principal` and `interestIndex` into one storage slot to save 1 SLOT (~2000 gas)

**Note: If within reason, the `uint` types of the struct fields can be altered in order for them to fit into one slot. For example, they could both have a `uint` type of `uint128`. Slight changes to the tests need to be done in order for this optimization to work. These changes should be as simple as changing the `uint` type for the struct fields. Since the tests are out of scope, this optimization is not included in the final diff, but is mentioned here for completeness**.
```solidity
File: contracts/VTokenInterfaces.sol
17:    struct BorrowSnapshot {
18:        uint256 principal;
19:        uint256 interestIndex;
20:    }
```
```diff
diff --git a/contracts/VTokenInterfaces.sol b/contracts/VTokenInterfaces.sol
index 8f578d7..b2296e0 100644
--- a/contracts/VTokenInterfaces.sol
+++ b/contracts/VTokenInterfaces.sol
@@ -15,8 +15,8 @@ contract VTokenStorage {
      * @member interestIndex Global borrowIndex as of the most recent balance-changing action
      */
     struct BorrowSnapshot {
-        uint256 principal;
-        uint256 interestIndex;
+        uint128 principal;
+        uint128 interestIndex;
     }
```

## State variables can be cached instead of re-reading them from storage

Total Instances: `40`

Estimated Gas Saved: `40 * 100 = 4000`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L60-L66

### Use already cached `assetsReserves[asset]` to save 1 SLOAD
```solidity
File: contracts/RiskFund/ReserveHelpers.sol
60:        uint256 assetReserve = assetsReserves[asset]; // @audit: 1st sload
61:        if (currentBalance > assetReserve) {
62:            uint256 balanceDifference;
63:            unchecked {
64:                balanceDifference = currentBalance - assetReserve;
65:            }
66:            assetsReserves[asset] += balanceDifference; // @audit: 2nd sload
```
```diff
diff --git a/contracts/RiskFund/ReserveHelpers.sol b/contracts/RiskFund/ReserveHelpers.sol
index 03aed00..d8c0b6e 100644
--- a/contracts/RiskFund/ReserveHelpers.sol
+++ b/contracts/RiskFund/ReserveHelpers.sol
@@ -63,9 +63,10 @@ contract ReserveHelpers {
             unchecked {
                 balanceDifference = currentBalance - assetReserve;
             }
-            assetsReserves[asset] += balanceDifference;
+            assetsReserves[asset] = assetReserve + balanceDifference;
             poolsAssetsReserves[comptroller][asset] += balanceDifference;
             emit AssetsReservesUpdated(comptroller, asset, balanceDifference);
         }
     }
 }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L72-L75

### Cache `poolsAssetsReserves[comptroller][asset]` to save 1 SLOAD
```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol
72:        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance"); // @audit: 1st sload
73:
74:        assetsReserves[asset] -= amount;
75:        poolsAssetsReserves[comptroller][asset] -= amount; // @audit: 2nd sload
```
```diff
diff --git a/contracts/RiskFund/ProtocolShareReserve.sol b/contracts/RiskFund/ProtocolShareReserve.sol
index a669b37..7238e37 100644
--- a/contracts/RiskFund/ProtocolShareReserve.sol
+++ b/contracts/RiskFund/ProtocolShareReserve.sol
@@ -69,10 +69,11 @@ contract ProtocolShareReserve is Ownable2StepUpgradeable, ExponentialNoError, Re
         uint256 amount
     ) external returns (uint256) {
         require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
-        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
+        uint256 _poolsAssetsReserves = poolsAssetsReserves[comptroller][asset];
+        require(amount <= _poolsAssetsReserves, "ProtocolShareReserve: Insufficient pool balance");

         assetsReserves[asset] -= amount;
-        poolsAssetsReserves[comptroller][asset] -= amount;
+        poolsAssetsReserves[comptroller][asset] = _poolsAssetsReserves - amount;
         uint256 protocolIncomeAmount = mul_(
             Exp({ mantissa: amount }),
             div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L192-L193

### Cache `poolReserves[comptroller]` to save 1 SLOAD
```solidity
File: contracts/RiskFund/RiskFund.sol
192:        require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");
193:        poolReserves[comptroller] = poolReserves[comptroller] - amount;
```
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..973ac9b 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -189,8 +189,9 @@ contract RiskFund is
      */
     function transferReserveForAuction(address comptroller, uint256 amount) external override returns (uint256) {
         require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");
-        require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");
-        poolReserves[comptroller] = poolReserves[comptroller] - amount;
+        uint256 _poolReserves = poolReserves[comptroller];
+        require(amount <= _poolReserves, "Risk Fund: Insufficient pool reserve.");
+        poolReserves[comptroller] = _poolReserves - amount;
         IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);

         emit TransferredReserveForAuction(comptroller, amount);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L405-L409

### Cache `incentiveBps` to save 1 SLOAD
```solidity
File: contracts/Shortfall/Shortfall.sol
405:        uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS); // @audit: 1st sload
406:        if (incentivizedRiskFundBalance >= riskFundBalance) {
407:            auction.startBidBps =
408:                (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
409:                (poolBadDebt * (MAX_BPS + incentiveBps)); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..fcbf7f3 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -402,19 +402,22 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         uint256 riskFundBalance = riskFund.poolReserves(comptroller);
         uint256 remainingRiskFundBalance = riskFundBalance;
-        uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
-        if (incentivizedRiskFundBalance >= riskFundBalance) {
-            auction.startBidBps =
-                (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
-                (poolBadDebt * (MAX_BPS + incentiveBps));
-            remainingRiskFundBalance = 0;
-            auction.auctionType = AuctionType.LARGE_POOL_DEBT;
-        } else {
-            uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;
+        {
+            uint256 _incentiveBps = incentiveBps;
+            uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * _incentiveBps) / MAX_BPS);
+            if (incentivizedRiskFundBalance >= riskFundBalance) {
+                auction.startBidBps =
+                    (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
+                    (poolBadDebt * (MAX_BPS + _incentiveBps));
+                remainingRiskFundBalance = 0;
+                auction.auctionType = AuctionType.LARGE_POOL_DEBT;
+            } else {
+                uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;

-            remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
-            auction.auctionType = AuctionType.LARGE_RISK_FUND;
-            auction.startBidBps = MAX_BPS;
+                remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
+                auction.auctionType = AuctionType.LARGE_RISK_FUND;
+                auction.startBidBps = MAX_BPS;
+            }
         }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L224-L236

### Use already cached `auction.markets[i]` to save 3 SLOADs
```solidity
File: contracts/Shortfall/Shortfall.sol
224:            VToken vToken = VToken(address(auction.markets[i])); // @audit: 1st sload
225:            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
226:
227:            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
228:                uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / MAX_BPS); // @audit: 2nd sload
229:                erc20.safeTransfer(address(auction.markets[i]), bidAmount); // @audit: 3rd sload
230:                marketsDebt[i] = bidAmount;
231:            } else {
232:                erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]); // @audit: 2nd & 3rd sload
233:                marketsDebt[i] = auction.marketDebt[auction.markets[i]]; // @audit: 4th sload
234:            }
235:
236:            auction.markets[i].badDebtRecovered(marketsDebt[i]); // @audit: 4th or 5th sload
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..d528881 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -225,15 +225,15 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

             if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
-                uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / MAX_BPS);
-                erc20.safeTransfer(address(auction.markets[i]), bidAmount);
+                uint256 bidAmount = ((auction.marketDebt[vToken] * auction.highestBidBps) / MAX_BPS);
+                erc20.safeTransfer(address(vToken), bidAmount);
                 marketsDebt[i] = bidAmount;
             } else {
-                erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);
-                marketsDebt[i] = auction.marketDebt[auction.markets[i]];
+                erc20.safeTransfer(address(vToken), auction.marketDebt[vToken]);
+                marketsDebt[i] = auction.marketDebt[vToken];
             }

-            auction.markets[i].badDebtRecovered(marketsDebt[i]);
+            vToken.badDebtRecovered(marketsDebt[i]);
         }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L214-L254

### Cache `auction.highestBidder`, `auction.auctionType`, and `auction.highestBidBps` to save 5 SLOADs
```solidity
File: contracts/Shortfall/Shortfall.sol
214:            block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0), // @audit: 1st sload
215:            "waiting for next bidder. cannot close auction"
216:        );
217:
218:        uint256 marketsCount = auction.markets.length;
219:        uint256[] memory marketsDebt = new uint256[](marketsCount);
220:
221:        auction.status = AuctionStatus.ENDED;
222:
223:        for (uint256 i; i < marketsCount; ++i) {
224:            VToken vToken = VToken(address(auction.markets[i]));
225:            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
226:
227:            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) { // @audit: 1st sload + on every iteration
228:                uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / MAX_BPS); // @audit: 1st sload + on every iteration
229:                erc20.safeTransfer(address(auction.markets[i]), bidAmount);
230:                marketsDebt[i] = bidAmount;
231:            } else {
232:                erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);
233:                marketsDebt[i] = auction.marketDebt[auction.markets[i]];
234:            }
235:
236:            auction.markets[i].badDebtRecovered(marketsDebt[i]);
237:        }
238:
239:        uint256 riskFundBidAmount;
240:
241:        if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) { // @audit: 2nd sload
242:            riskFundBidAmount = auction.seizedRiskFund;
243:        } else {
244:            riskFundBidAmount = (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS; // @audit: 2nd sload
245:        }
246:
247:        uint256 transferredAmount = riskFund.transferReserveForAuction(comptroller, riskFundBidAmount);
248:        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(auction.highestBidder, riskFundBidAmount); // @audit: 2nd sload
249:
250:        emit AuctionClosed(
251:            comptroller,
252:            auction.startBlock,
253:            auction.highestBidder, // @audit: 3rd sload
254:            auction.highestBidBps, // @audit: 3rd sload
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..8b14530 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -210,8 +210,9 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         Auction storage auction = auctions[comptroller];

         require(_isStarted(auction), "no on-going auction");
+        address _highestBidder = auction.highestBidder;
         require(
-            block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),
+            block.number > auction.highestBidBlock + nextBidderBlockLimit && _highestBidder != address(0),
             "waiting for next bidder. cannot close auction"
         );

@@ -219,13 +220,15 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         uint256[] memory marketsDebt = new uint256[](marketsCount);

         auction.status = AuctionStatus.ENDED;
-
+
+        AuctionType _auctionType = auction.auctionType;
+        uint256 _highestBidBps = auction.highestBidBps;
         for (uint256 i; i < marketsCount; ++i) {
             VToken vToken = VToken(address(auction.markets[i]));
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

-            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
-                uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / MAX_BPS);
+            if (_auctionType == AuctionType.LARGE_POOL_DEBT) {
+                uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * _highestBidBps) / MAX_BPS);
                 erc20.safeTransfer(address(auction.markets[i]), bidAmount);
                 marketsDebt[i] = bidAmount;
             } else {
@@ -238,20 +241,20 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         uint256 riskFundBidAmount;

-        if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
+        if (_auctionType == AuctionType.LARGE_POOL_DEBT) {
             riskFundBidAmount = auction.seizedRiskFund;
         } else {
-            riskFundBidAmount = (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS;
+            riskFundBidAmount = (auction.seizedRiskFund * _highestBidBps) / MAX_BPS;
         }

         uint256 transferredAmount = riskFund.transferReserveForAuction(comptroller, riskFundBidAmount);
-        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(auction.highestBidder, riskFundBidAmount);
+        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(_highestBidder, riskFundBidAmount);

         emit AuctionClosed(
             comptroller,
             auction.startBlock,
-            auction.highestBidder,
-            auction.highestBidBps,
+            _highestBidder,
+            _highestBidBps,
             transferredAmount,
             auction.markets,
             marketsDebt
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L176-L194

### Use already cached `auction.markets[i]` and cache `auction.marketDebt[auction.markets[i]]` to save 3 SLOADs
```solidity
File: contracts/Shortfall/Shortfall.sol
176:            VToken vToken = VToken(address(auction.markets[i])); // @audit: 1st sload
177:            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
178:
179:            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
180:                if (auction.highestBidder != address(0)) {
181:                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / // @audit: 2nd sload & 1st sload
182:                        MAX_BPS);
183:                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
184:                }
185:
186:                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS); // @audit: 3rd sload & 2nd sload
187:                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
188:            } else {
189:                if (auction.highestBidder != address(0)) {
190:                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]); // @audit: 2nd sload & 1st sload
191:                }
192:
193:                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]); // @audit: 3rd sload & 2nd sload
194:            }
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..b00060d 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -177,20 +177,22 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

             if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
+                uint256 _marketDebt = auction.marketDebt[vToken];
                 if (auction.highestBidder != address(0)) {
-                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
+                    uint256 previousBidAmount = ((_marketDebt * auction.highestBidBps) /
                         MAX_BPS);
                     erc20.safeTransfer(auction.highestBidder, previousBidAmount);
                 }

-                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
+                uint256 currentBidAmount = ((_marketDebt * bidBps) / MAX_BPS);
                 erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
             } else {
+                uint256 _marketDebt = auction.marketDebt[vToken];
                 if (auction.highestBidder != address(0)) {
-                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
+                    erc20.safeTransfer(auction.highestBidder, _marketDebt);
                 }

-                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
+                erc20.safeTransferFrom(msg.sender, address(this), _marketDebt);
             }
         }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L164-L191

### Cache `auction.auctionType`, `auction.highestBidder`, and `auction.highestBidBps` to save 9 SLOADs
```solidity
File: contracts/Shortfall/Shortfall.sol
164:        require(
165:            (auction.auctionType == AuctionType.LARGE_POOL_DEBT && // @audit: 1st sload
166:                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) || // @audit: 1st sload & 1st sload
167:                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) || // @audit: 2nd sload
168:                (auction.auctionType == AuctionType.LARGE_RISK_FUND && // @audit: 2nd sload
169:                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) || // @audit: 3rd sload & 2nd sload
170:                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))), // @audit: 4th sload
171:            "your bid is not the highest"
172:        );
173:
174:        uint256 marketsCount = auction.markets.length;
175:        for (uint256 i; i < marketsCount; ++i) {
176:            VToken vToken = VToken(address(auction.markets[i]));
177:            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
178:
179:            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) { // @audit: 3rd sload + on every iteration
180:                if (auction.highestBidder != address(0)) { // @audit: 5th sload + on every iteration
181:                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) // @audit: 3rd sload + on every iteration
182:                        MAX_BPS);
183:                    erc20.safeTransfer(auction.highestBidder, previousBidAmount); // @audit: 6th sload + on every iteration
184:                }
185:
186:                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
187:                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
188:            } else {
189:                if (auction.highestBidder != address(0)) { // @audit: 5th sload + on every iteration
190:                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]); // @audit: 6th sload + on every iteration
191:                } 
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..1e09bcc 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -161,13 +161,16 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         require(_isStarted(auction), "no on-going auction");
         require(!_isStale(auction), "auction is stale, restart it");
         require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
+        AuctionType _auctionType = auction.auctionType;
+        address _highestBidder = auction.highestBidder;
+        uint256 _highestBidBps = auction.highestBidBps;
         require(
-            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
-                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
-                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
-                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
-                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
-                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
+            (_auctionType == AuctionType.LARGE_POOL_DEBT &&
+                ((_highestBidder != address(0) && bidBps > _highestBidBps) ||
+                    (_highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
+                (_auctionType == AuctionType.LARGE_RISK_FUND &&
+                    ((_highestBidder != address(0) && bidBps < _highestBidBps) ||
+                        (_highestBidder == address(0) && bidBps <= auction.startBidBps))),
             "your bid is not the highest"
         );

@@ -176,18 +179,18 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             VToken vToken = VToken(address(auction.markets[i]));
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

-            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
-                if (auction.highestBidder != address(0)) {
-                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
+            if (_auctionType == AuctionType.LARGE_POOL_DEBT) {
+                if (_highestBidder != address(0)) {
+                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * _highestBidBps) /
                         MAX_BPS);
-                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
+                    erc20.safeTransfer(_highestBidder, previousBidAmount);
                 }

                 uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                 erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
             } else {
-                if (auction.highestBidder != address(0)) {
-                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
+                if (_highestBidder != address(0)) {
+                    erc20.safeTransfer(_highestBidder, auction.marketDebt[auction.markets[i]]);
                 }

                 erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L213-L229

### Use already cached `accountAssets[msg.sender].length` to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
213:        VToken[] memory userAssetList = accountAssets[msg.sender];
214:        uint256 len = userAssetList.length; // @audit: 1st sload
... 
227:        // copy last item in list to location of item to be removed, reduce length by 1
228:        VToken[] storage storedList = accountAssets[msg.sender];
229:        storedList[assetIndex] = storedList[storedList.length - 1]; // @audit: 2nd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..f15da10 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -226,7 +226,7 @@ contract Comptroller is

         // copy last item in list to location of item to be removed, reduce length by 1
         VToken[] storage storedList = accountAssets[msg.sender];
-        storedList[assetIndex] = storedList[storedList.length - 1];
+        storedList[assetIndex] = storedList[len - 1];
         storedList.pop();

         emit MarketExited(vToken, msg.sender);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L275-L276

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
275:            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken); // @audit: 1st sload
276:            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..4f1f429 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -272,8 +272,9 @@ contract Comptroller is
         uint256 rewardDistributorsCount = rewardsDistributors.length;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
-            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenSupplyIndex(vToken);
+            _rewardsDistributor.distributeSupplierRewardToken(vToken, minter);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L305-L306

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
305:            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken); // @audit: 1st sload
306:            rewardsDistributors[i].distributeSupplierRewardToken(vToken, redeemer); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..7c7c91a 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -302,8 +302,9 @@ contract Comptroller is
         uint256 rewardDistributorsCount = rewardsDistributors.length;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
-            rewardsDistributors[i].distributeSupplierRewardToken(vToken, redeemer);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenSupplyIndex(vToken);
+            _rewardsDistributor.distributeSupplierRewardToken(vToken, redeemer);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L377-L378

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
377:            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex); // @audit: 1st sload
378:            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..8d987c5 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -374,8 +374,9 @@ contract Comptroller is
         uint256 rewardDistributorsCount = rewardsDistributors.length;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
-            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenBorrowIndex(vToken, borrowIndex);
+            _rewardsDistributor.distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L404-L405

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
404:            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex); // @audit: 1st sload
405:            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex); // @audit: 2nd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..927183d 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -401,8 +401,9 @@ contract Comptroller is

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
-            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
-            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenBorrowIndex(vToken, borrowIndex);
+            _rewardsDistributor.distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L522-L524

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 2 SLOADs
```solidity
File: contracts/Comptroller.sol
522:            rewardsDistributors[i].updateRewardTokenSupplyIndex(vTokenCollateral); // @audit: 1st sload
523:            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, borrower); // @audit: 2nd sload
524:            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, liquidator); // @audit: 3rd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..24dcb75 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -519,9 +519,10 @@ contract Comptroller is
         uint256 rewardDistributorsCount = rewardsDistributors.length;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            rewardsDistributors[i].updateRewardTokenSupplyIndex(vTokenCollateral);
-            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, borrower);
-            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, liquidator);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenSupplyIndex(vTokenCollateral);
+            _rewardsDistributor.distributeSupplierRewardToken(vTokenCollateral, borrower);
+            _rewardsDistributor.distributeSupplierRewardToken(vTokenCollateral, liquidator);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L559-L561

**Note: The automated report gave the wrong description and explanation for these lines of code. They are included here for completeness, but will not be included in final diffs.**

### Cache `rewardsDistributors[i]` to save 2 SLOADs
```solidity
File: contracts/Comptroller.sol
559:            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken); // @audit: 1st sload
560:            rewardsDistributors[i].distributeSupplierRewardToken(vToken, src); // @audit: 2nd sload
561:            rewardsDistributors[i].distributeSupplierRewardToken(vToken, dst); // @audit: 3rd sload
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..bf403d2 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -556,9 +556,10 @@ contract Comptroller is
         uint256 rewardDistributorsCount = rewardsDistributors.length;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
-            rewardsDistributors[i].distributeSupplierRewardToken(vToken, src);
-            rewardsDistributors[i].distributeSupplierRewardToken(vToken, dst);
+            RewardsDistributor _rewardsDistributor = rewardsDistributors[i];
+            _rewardsDistributor.updateRewardTokenSupplyIndex(vToken);
+            _rewardsDistributor.distributeSupplierRewardToken(vToken, src);
+            _rewardsDistributor.distributeSupplierRewardToken(vToken, dst);
         }
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1206-L1214

### Use `marketsCount + 1` instead of re-reading from storage to save 1 SLOAD
```solidity
File: contracts/Comptroller.sol
1206:        uint256 marketsCount = allMarkets.length;
1207:
1208:        for (uint256 i; i < marketsCount; ++i) {
1209:            if (allMarkets[i] == VToken(vToken)) {
1210:                revert MarketAlreadyListed(vToken);
1211:            }
1212:        }
1213:        allMarkets.push(VToken(vToken));
1214:        marketsCount = allMarkets.length; // @audit: unnecessary sload. New length is `marketsCount + 1`
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..cb7a2ef 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -1211,7 +1211,7 @@ contract Comptroller is
             }
         }
         allMarkets.push(VToken(vToken));
-        marketsCount = allMarkets.length;
+        marketsCount += 1;
         _ensureMaxLoops(marketsCount);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L930-L941

### Use alread cached `rewardsDistributors.length` instead of re-reading from storage
```solidity
File: contracts/Comptroller.sol
930:        uint256 rewardsDistributorsLength = rewardsDistributors.length;
931:
932:        for (uint256 i; i < rewardsDistributorsLength; ++i) {
933:            address rewardToken = address(rewardsDistributors[i].rewardToken());
934:            require(
935:                rewardToken != address(_rewardsDistributor.rewardToken()),
936:                "distributor already exists with this reward"
937:            );
938:        }
939:
940:        uint256 rewardsDistributorsLen = rewardsDistributors.length;
941:        _ensureMaxLoops(rewardsDistributorsLen + 1);
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..ad749b5 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -937,8 +937,7 @@ contract Comptroller is
             );
         }

-        uint256 rewardsDistributorsLen = rewardsDistributors.length;
-        _ensureMaxLoops(rewardsDistributorsLen + 1);
+        _ensureMaxLoops(rewardsDistributorsLength + 1);

         rewardsDistributors.push(_rewardsDistributor);
         rewardsDistributorExists[address(_rewardsDistributor)] = true;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L490-L492

### Cache `badDebt` to save 1 SLOAD
```solidity
File: contracts/VToken.sol
490:        require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction"); // @audit: 1st sload
491:
492:        uint256 badDebtOld = badDebt; // @audit: 2nd sload
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..67d6536 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -487,9 +487,9 @@ contract VToken is
      */
     function badDebtRecovered(uint256 recoveredAmount_) external {
         require(msg.sender == shortfall, "only shortfall contract can update bad debt");
-        require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");
-
         uint256 badDebtOld = badDebt;
+        require(recoveredAmount_ <= badDebtOld, "more than bad debt recovered from auction");
+
         uint256 badDebtNew = badDebtOld - recoveredAmount_;
         badDebt = badDebtNew;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1026-L1067

### Cache `comptroller` to save 1 SLOAD
```solidity
File: contracts/VToken.sol
1026:        comptroller.preLiquidateHook( // @audit: 1st sload
1027:            address(this),
1028:            address(vTokenCollateral),
1029:            borrower,
1030:            repayAmount,
1031:            skipLiquidityCheck
1032:        );
...
1066:        /* We calculate the number of collateral tokens that will be seized */
1067:        (uint256 amountSeizeError, uint256 seizeTokens) = comptroller.liquidateCalculateSeizeTokens( // @audit: 2nd sload
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..ffbc0cf 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1023,7 +1023,8 @@ contract VToken is
         bool skipLiquidityCheck
     ) internal {
         /* Fail if liquidate not allowed */
-        comptroller.preLiquidateHook(
+        ComptrollerInterface _comptroller = comptroller;
+        _comptroller.preLiquidateHook(
             address(this),
             address(vTokenCollateral),
             borrower,
@@ -1064,7 +1065,7 @@ contract VToken is
         // (No safe failures beyond this point)

         /* We calculate the number of collateral tokens that will be seized */
-        (uint256 amountSeizeError, uint256 seizeTokens) = comptroller.liquidateCalculateSeizeTokens(
+        (uint256 amountSeizeError, uint256 seizeTokens) = _comptroller.liquidateCalculateSeizeTokens(
             address(this),
             address(vTokenCollateral),
             actualRepayAmount
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1215-L1235

### Cache `totalReserves` and `protocolShareReserve` to save 3 SLOADs
```solidity
1215:        if (reduceAmount > totalReserves) { // @audit: 1st sload
1216:            revert ReduceReservesCashValidation();
1217:        }
1218:
1219:        /////////////////////////
1220:        // EFFECTS & INTERACTIONS
1221:        // (No safe failures beyond this point)
1222:
1223:        totalReservesNew = totalReserves - reduceAmount; // @audit: 2nd sload
1224:
1225:        // Store reserves[n+1] = reserves[n] - reduceAmount
1226:        totalReserves = totalReservesNew;
1227:
1228:        // _doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
1229:        // Transferring an underlying asset to the protocolShareReserve contract to channel the funds for different use.
1230:        _doTransferOut(protocolShareReserve, reduceAmount); // @audit: 1st sload
1231:
1232:        // Update the pool asset's state in the protocol share reserve for the above transfer.
1233:        IProtocolShareReserve(protocolShareReserve).updateAssetsState(address(comptroller), underlying); // @audit: 2nd sload
1234:
1235:        emit ReservesReduced(protocolShareReserve, reduceAmount, totalReservesNew); // @audit: 3rd sload
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..1d4a867 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1212,7 +1212,8 @@ contract VToken is
         }

         // Check reduceAmount ≤ reserves[n] (totalReserves)
-        if (reduceAmount > totalReserves) {
+        uint256 _totalReserves = totalReserves;
+        if (reduceAmount > _totalReserves) {
             revert ReduceReservesCashValidation();
         }

@@ -1220,19 +1221,20 @@ contract VToken is
         // EFFECTS & INTERACTIONS
         // (No safe failures beyond this point)

-        totalReservesNew = totalReserves - reduceAmount;
+        totalReservesNew = _totalReserves - reduceAmount;

         // Store reserves[n+1] = reserves[n] - reduceAmount
         totalReserves = totalReservesNew;

         // _doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
         // Transferring an underlying asset to the protocolShareReserve contract to channel the funds for different use.
-        _doTransferOut(protocolShareReserve, reduceAmount);
+        address _protocolShareReserve = protocolShareReserve;
+        _doTransferOut(_protocolShareReserve, reduceAmount);

         // Update the pool asset's state in the protocol share reserve for the above transfer.
-        IProtocolShareReserve(protocolShareReserve).updateAssetsState(address(comptroller), underlying);
+        IProtocolShareReserve(_protocolShareReserve).updateAssetsState(address(comptroller), underlying);

-        emit ReservesReduced(protocolShareReserve, reduceAmount, totalReservesNew);
+        emit ReservesReduced(_protocolShareReserve, reduceAmount, totalReservesNew);
     }
```

## Cache state variables outside of loop to avoid reading storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L282-L284

*Gas Savings for `RewardsDistributor.claimRewardToken`, obtained via protocol's tests: Avg 77 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  262655  |  291169  |  276621 |    3   |
| After  |  262578  |  291092  |  276544 |    3   |

```solidity
File: contracts/Rewards/RewardsDistributor.sol
282:        for (uint256 i; i < vTokensCount; ++i) {
283:            VToken vToken = vTokens[i];
284:            require(comptroller.isMarketListed(vToken), "market must be listed");
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..1b92e43 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -278,10 +278,11 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
         uint256 vTokensCount = vTokens.length;

         _ensureMaxLoops(vTokensCount);
-
+
+        Comptroller _comptroller = comptroller;
         for (uint256 i; i < vTokensCount; ++i) {
             VToken vToken = vTokens[i];
-            require(comptroller.isMarketListed(vToken), "market must be listed");
+            require(_comptroller.isMarketListed(vToken), "market must be listed");
             Exp memory borrowIndex = Exp({ mantissa: vToken.borrowIndex() });
             _updateRewardTokenBorrowIndex(address(vToken), borrowIndex);
             _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L584-L586

*Gas Savings for `Comptroller.healAccount`, obtained via protocol's tests: Avg 181 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  84584   |  331091  |  223131 |    10   |
| After  |  84324   |  330963  |  222950 |    10   |

### Cache `oracle` outside loop to save 1 SLOAD per iteration. 

*Note: we must remove cached `msg.sender` variable to fix `stack too deep` error*.
```solidity
File: contracts/Comptroller.sol
584:        for (uint256 i; i < userAssetsCount; ++i) {
585:            userAssets[i].accrueInterest();
586:            oracle.updatePrice(address(userAssets[i])); // @audit: sload on every iteration
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..8ae7709 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -579,12 +579,13 @@ contract Comptroller is
         VToken[] memory userAssets = accountAssets[user];
         uint256 userAssetsCount = userAssets.length;

-        address liquidator = msg.sender;
         // We need all user's markets to be fresh for the computations to be correct
+        PriceOracle _oracle = oracle;
         for (uint256 i; i < userAssetsCount; ++i) {
             userAssets[i].accrueInterest();
-            oracle.updatePrice(address(userAssets[i]));
+            _oracle.updatePrice(address(userAssets[i]));
         }
+

         AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(user, _getLiquidationThreshold);

@@ -616,11 +617,11 @@ contract Comptroller is

             // Seize the entire collateral
             if (tokens != 0) {
-                market.seize(liquidator, user, tokens);
+                market.seize(msg.sender, user, tokens);
             }
             // Repay a certain percentage of the borrow, forgive the rest
             if (borrowBalance != 0) {
-                market.healBorrow(liquidator, user, repaymentAmount);
+                market.healBorrow(msg.sender, user, repaymentAmount);
             }
         }
     }
```

## Avoid emitting storage values
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L157-L162

### Cache values given to storage variables and emit cached values + `kink_` to save 4 SLOADs
```solidity
File: contracts/BaseJumpRateModelV2.sol
157:        baseRatePerBlock = baseRatePerYear / blocksPerYear; // @audit: cache value 
158:        multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_); // @audit: cache value 
159:        jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear; // @audit: cache value 
160:        kink = kink_;
161:
162:        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink); // @audit: emit cached values and `kink_`
```
```diff
diff --git a/contracts/BaseJumpRateModelV2.sol b/contracts/BaseJumpRateModelV2.sol
index 68b535a..0b51743 100644
--- a/contracts/BaseJumpRateModelV2.sol
+++ b/contracts/BaseJumpRateModelV2.sol
@@ -154,12 +154,15 @@ abstract contract BaseJumpRateModelV2 is InterestRateModel {
         uint256 jumpMultiplierPerYear,
         uint256 kink_
     ) internal {
-        baseRatePerBlock = baseRatePerYear / blocksPerYear;
-        multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
-        jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
+        uint256 _baseRatePerBlock = baseRatePerYear / blocksPerYear;
+        baseRatePerBlock = _baseRatePerBlock;
+        uint256 _multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
+        multiplierPerBlock = _multiplierPerBlock;
+        uint256 _jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
+        jumpMultiplierPerBlock = _jumpMultiplierPerBlock;
         kink = kink_;

-        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
+        emit NewInterestParams(_baseRatePerBlock, _multiplierPerBlock, _jumpMultiplierPerBlock, kink_);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L421-L427

### Emit `block.timestamp` instead of reading from storage to save 1 SLOAD
```solidity
File: contracts/Shortfall/Shortfall.sol
421:        auction.startBlock = block.number;
422:        auction.status = AuctionStatus.STARTED;
423:        auction.highestBidder = address(0);
424:
425:        emit AuctionStarted(
426:            comptroller,
427:            auction.startBlock, // @audit: emit block.timestamp
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..89474e4 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -424,7 +424,7 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         emit AuctionStarted(
             comptroller,
-            auction.startBlock,
+            block.timestamp,
             auction.auctionType,
             auction.markets,
             marketsDebt,
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L420-L431

### Cache expression and emit cached value to save 1 SLOAD
```solidity
File: contracts/Shortfall/Shortfall.sol
420:        auction.seizedRiskFund = riskFundBalance - remainingRiskFundBalance; // @audit: cache expression
421:        auction.startBlock = block.number;
422:        auction.status = AuctionStatus.STARTED;
423:        auction.highestBidder = address(0);
424:
425:        emit AuctionStarted(
426:            comptroller,
427:            auction.startBlock,
428:            auction.auctionType,
429:            auction.markets,
430:            marketsDebt,
431:            auction.seizedRiskFund, // @audit: cache expression (`riskFundBalance - remainingRiskFundBalance`) and emit expression
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..263724b 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -402,22 +402,25 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         uint256 riskFundBalance = riskFund.poolReserves(comptroller);
         uint256 remainingRiskFundBalance = riskFundBalance;
-        uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
-        if (incentivizedRiskFundBalance >= riskFundBalance) {
-            auction.startBidBps =
-                (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
-                (poolBadDebt * (MAX_BPS + incentiveBps));
-            remainingRiskFundBalance = 0;
-            auction.auctionType = AuctionType.LARGE_POOL_DEBT;
-        } else {
-            uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;
+        {
+            uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
+            if (incentivizedRiskFundBalance >= riskFundBalance) {
+                auction.startBidBps =
+                    (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
+                    (poolBadDebt * (MAX_BPS + incentiveBps));
+                remainingRiskFundBalance = 0;
+                auction.auctionType = AuctionType.LARGE_POOL_DEBT;
+            } else {
+                uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;

-            remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
-            auction.auctionType = AuctionType.LARGE_RISK_FUND;
-            auction.startBidBps = MAX_BPS;
+                remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
+                auction.auctionType = AuctionType.LARGE_RISK_FUND;
+                auction.startBidBps = MAX_BPS;
+            }
         }
-
-        auction.seizedRiskFund = riskFundBalance - remainingRiskFundBalance;
+
+        uint256 _seizeRiskFund = riskFundBalance - remainingRiskFundBalance;
+        auction.seizedRiskFund = _seizeRiskFund;
         auction.startBlock = block.number;
         auction.status = AuctionStatus.STARTED;
         auction.highestBidder = address(0);
@@ -428,7 +431,7 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             auction.auctionType,
             auction.markets,
             marketsDebt,
-            auction.seizedRiskFund,
+            _seizeRiskFund,
             auction.startBidBps
         );
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L407-L432

### Cache values given to `auction.startBidBps` & `auction.auctionType` and emit cached values to save 2 SLOADs
```solidity
File: contracts/Shortfall/Shortfall.sol
407:            auction.startBidBps =
408:                (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
409:                (poolBadDebt * (MAX_BPS + incentiveBps));
410:            remainingRiskFundBalance = 0;
411:            auction.auctionType = AuctionType.LARGE_POOL_DEBT;
412:        } else {
413:            uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;
414:
415:            remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
416:            auction.auctionType = AuctionType.LARGE_RISK_FUND;
417:            auction.startBidBps = MAX_BPS;
418:        }
...
425:        emit AuctionStarted(
426:            comptroller,
427:            auction.startBlock,
428:            auction.auctionType, // @audit: cache value previously given to state variable and emit cached value
429:            auction.markets,
430:            marketsDebt,
431:            auction.seizedRiskFund,
432:            auction.startBidBps // @audit: cache value previously given to state variable and emit cached value
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..6d8bb6d 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -399,37 +399,43 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         }

         require(poolBadDebt >= minimumPoolBadDebt, "pool bad debt is too low");
-
+
+        uint256 _startBidBps;
+        AuctionType _auctionType;
         uint256 riskFundBalance = riskFund.poolReserves(comptroller);
         uint256 remainingRiskFundBalance = riskFundBalance;
-        uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
-        if (incentivizedRiskFundBalance >= riskFundBalance) {
-            auction.startBidBps =
-                (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
-                (poolBadDebt * (MAX_BPS + incentiveBps));
-            remainingRiskFundBalance = 0;
-            auction.auctionType = AuctionType.LARGE_POOL_DEBT;
-        } else {
-            uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;
+        {
+            uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
+            if (incentivizedRiskFundBalance >= riskFundBalance) {
+                _startBidBps =
+                    (MAX_BPS * MAX_BPS * remainingRiskFundBalance) /
+                    (poolBadDebt * (MAX_BPS + incentiveBps));
+                remainingRiskFundBalance = 0;
+                _auctionType = AuctionType.LARGE_POOL_DEBT;
+            } else {
+                uint256 maxSeizeableRiskFundBalance = incentivizedRiskFundBalance;

-            remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
-            auction.auctionType = AuctionType.LARGE_RISK_FUND;
-            auction.startBidBps = MAX_BPS;
+                remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
+                _auctionType = AuctionType.LARGE_RISK_FUND;
+                _startBidBps = MAX_BPS;
+            }
         }

         auction.seizedRiskFund = riskFundBalance - remainingRiskFundBalance;
         auction.startBlock = block.number;
         auction.status = AuctionStatus.STARTED;
         auction.highestBidder = address(0);
+        auction.startBidBps = _startBidBps;
+        auction.auctionType = _auctionType;

         emit AuctionStarted(
             comptroller,
             auction.startBlock,
-            auction.auctionType,
+            _auctionType,
             auction.markets,
             marketsDebt,
             auction.seizedRiskFund,
-            auction.startBidBps
+            _startBidBps
         );
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L263-L268

### Emit `contributorAccrued` to save 1 SLOAD
```solidity
File: contracts/Rewards/RewardsDistributor.sol
263:            uint256 contributorAccrued = add_(rewardTokenAccrued[contributor], newAccrued);
264:
265:            rewardTokenAccrued[contributor] = contributorAccrued;
266:            lastContributorBlock[contributor] = blockNumber;
267:
268:            emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]); // @audit: emit `contributorAccrued`
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..32eb079 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -265,7 +265,7 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
             rewardTokenAccrued[contributor] = contributorAccrued;
             lastContributorBlock[contributor] = blockNumber;

-            emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
+            emit ContributorRewardsUpdated(contributor, contributorAccrued);
         }
     }
```

## Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L343

*Gas Savings for `PoolRegistry.updatePoolMetadata`, obtained via protocol's tests: Avg 747 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  115061   |  115109  |  115095 |    5   |
| After  |  114314   |  114362  |  114348 |    5   |

```solidity
File: contracts/Pool/PoolRegistry.sol
343:    function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {
```
```diff
diff --git a/contracts/Pool/PoolRegistry.sol b/contracts/Pool/PoolRegistry.sol
index 5cf376f..72c1800 100644
--- a/contracts/Pool/PoolRegistry.sol
+++ b/contracts/Pool/PoolRegistry.sol
@@ -340,7 +340,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
     /**
      * @notice Update metadata of an existing pool.
      */
-    function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {
+    function updatePoolMetadata(address comptroller, VenusPoolMetaData calldata _metadata) external {
         _checkAccessAllowed("updatePoolMetadata(address,VenusPoolMetaData)");
         VenusPoolMetaData memory oldMetadata = metadata[comptroller];
         metadata[comptroller] = _metadata;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L197-L201

*Gas Savings for `RewardsDistributor.setRewardTokenSpeeds`, obtained via protocol's tests: Avg 844 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  208377 |    20   |
| After  |  -   |  -  |  207533 |    20   |

```solidity
File: contracts/Rewards/RewardsDistributor.sol
197:    function setRewardTokenSpeeds(
198:        VToken[] memory vTokens,
199:        uint256[] memory supplySpeeds,
200:        uint256[] memory borrowSpeeds
201:    ) external {
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..2b5da9f 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -195,9 +195,9 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
      * @param borrowSpeeds New borrow-side REWARD TOKEN speed for the corresponding market.
      */
     function setRewardTokenSpeeds(
-        VToken[] memory vTokens,
-        uint256[] memory supplySpeeds,
-        uint256[] memory borrowSpeeds
+        VToken[] calldata vTokens,
+        uint256[] calldata supplySpeeds,
+        uint256[] calldata borrowSpeeds
     ) external {
         _checkAccessAllowed("setRewardTokenSpeeds(address[],uint256[],uint256[])");
         uint256 numTokens = vTokens.length;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154

*Gas Savings for `Comptroller.enterMarkets`, obtained via protocol's tests: Avg 134 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  41896   |  209652  |  132444 |    110   |
| After  |  41772   |  209500  |  132310 |    110   |

```solidity
File: contracts/Comptroller.sol
154:    function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..d9e318d 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -151,7 +151,7 @@ contract Comptroller is
      * @custom:error MarketNotListed error is thrown if any of the markets is not listed
      * @custom:access Not restricted
      */
-    function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {
+    function enterMarkets(address[] calldata vTokens) external override returns (uint256[] memory) {
         uint256 len = vTokens.length;

         uint256 accountAssetsLen = accountAssets[msg.sender].length;
```

## Refactor internal function to avoid unnecessary SLOAD
The internal functions below read storage slots that are previously read in the functions that invoke them. We can refactor the internal functions so we could pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L304-L308

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L209-L210

*Gas Savings for `Shortfall.setRewardTokenSpeeds`, obtained via protocol's tests: Avg 77 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  208377 |    20   |
| After  |  -   |  -  |  208300 |    20   |

```solidity
File: contracts/Rewards/RewardsDistributor.sol
304:    function _setRewardTokenSpeed(
305:        VToken vToken,
306:        uint256 supplySpeed,
307:        uint256 borrowSpeed
308:    ) internal {

209:        for (uint256 i; i < numTokens; ++i) {
210:            _setRewardTokenSpeed(vTokens[i], supplySpeeds[i], borrowSpeeds[i]); // @audit: only time `_setRewardTokenSpeed` is used
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..b587dff 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -205,9 +205,10 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
             numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,
             "RewardsDistributor::setRewardTokenSpeeds invalid input"
         );
-
+
+        Comptroller _comptroller = comptroller;
         for (uint256 i; i < numTokens; ++i) {
-            _setRewardTokenSpeed(vTokens[i], supplySpeeds[i], borrowSpeeds[i]);
+            _setRewardTokenSpeed(_comptroller, vTokens[i], supplySpeeds[i], borrowSpeeds[i]);
         }
     }

@@ -302,11 +303,12 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
      * @param borrowSpeed New borrow-side REWARD TOKEN speed for market
      */
     function _setRewardTokenSpeed(
+        Comptroller _comptroller,
         VToken vToken,
         uint256 supplySpeed,
         uint256 borrowSpeed
     ) internal {
-        require(comptroller.isMarketListed(vToken), "rewardToken market is not listed");
+        require(_comptroller.isMarketListed(vToken), "rewardToken market is not listed");

         if (rewardTokenSupplySpeeds[address(vToken)] != supplySpeed) {
             // Supply speed updated so let's update supply state to ensure that
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L458-L460

*Gas Savings for `Shortfall.restartAuction`, obtained via protocol's tests: Avg 100 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  114761 |    2   |
| After  |  -   |  -  |  114661 |    2   |

```solidity
File: contracts/Shortfall/Shortfall.sol
458:    function _isStarted(Auction storage auction) internal view returns (bool) {
459:        return auction.startBlock != 0 && auction.status == AuctionStatus.STARTED;
460:    }
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..d237bb2 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -157,8 +157,9 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
      */
     function placeBid(address comptroller, uint256 bidBps) external nonReentrant {
         Auction storage auction = auctions[comptroller];
-
-        require(_isStarted(auction), "no on-going auction");
+
+        uint256 _startBlock = auction.startBlock;
+        require(_isStarted(_startBlock, auction), "no on-going auction");
         require(!_isStale(auction), "auction is stale, restart it");
         require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
         require(
@@ -198,7 +199,7 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         auction.highestBidBps = bidBps;
         auction.highestBidBlock = block.number;

-        emit BidPlaced(comptroller, auction.startBlock, bidBps, msg.sender);
+        emit BidPlaced(comptroller, _startBlock, bidBps, msg.sender);
     }

     /**
@@ -208,8 +209,9 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
      */
     function closeAuction(address comptroller) external nonReentrant {
         Auction storage auction = auctions[comptroller];
-
-        require(_isStarted(auction), "no on-going auction");
+
+        uint256 _startBlock = auction.startBlock;
+        require(_isStarted(_startBlock, auction), "no on-going auction");
         require(
             block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),
             "waiting for next bidder. cannot close auction"
@@ -249,7 +251,7 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         emit AuctionClosed(
             comptroller,
-            auction.startBlock,
+            _startBlock,
             auction.highestBidder,
             auction.highestBidBps,
             transferredAmount,
@@ -274,13 +276,14 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
      */
     function restartAuction(address comptroller) external {
         Auction storage auction = auctions[comptroller];
-
-        require(_isStarted(auction), "no on-going auction");
+
+        uint256 _startBlock = auction.startBlock;
+        require(_isStarted(_startBlock, auction), "no on-going auction");
         require(_isStale(auction), "you need to wait for more time for first bidder");

         auction.status = AuctionStatus.ENDED;

-        emit AuctionRestarted(comptroller, auction.startBlock);
+        emit AuctionRestarted(comptroller, _startBlock);
         _startAuction(comptroller);
     }

@@ -455,8 +458,8 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
      * @dev Checks if the auction has started
      * @param auction The auction to query the status for
      */
-    function _isStarted(Auction storage auction) internal view returns (bool) {
-        return auction.startBlock != 0 && auction.status == AuctionStatus.STARTED;
+    function _isStarted(uint256 _startBlock, Auction storage auction) internal view returns (bool) {
+        return _startBlock != 0 && auction.status == AuctionStatus.STARTED;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L467-L470

*Gas Savings for `Shortfall.placeBid`, obtained via protocol's tests: Avg 633 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  140790   |  216414  |  185429 |    5   |
| After  |  140055   |  215834  |  184796 |    5   |

```solidity
File: contracts/Shortfall/Shortfall.sol
467:    function _isStale(Auction storage auction) internal view returns (bool) {
468:        bool noBidder = auction.highestBidder == address(0);
469:        return noBidder && (block.number > auction.startBlock + waitForFirstBidder);
470:    }
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..a8b7bf5 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -159,15 +159,17 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         Auction storage auction = auctions[comptroller];

         require(_isStarted(auction), "no on-going auction");
-        require(!_isStale(auction), "auction is stale, restart it");
+        uint256 _startBlock = auction.startBlock;
+        address _highestBidder = auction.highestBidder;
+        require(!_isStale(_startBlock, _highestBidder), "auction is stale, restart it");
         require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
         require(
             (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
-                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
-                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
+                ((_highestBidder != address(0) && bidBps > auction.highestBidBps) ||
+                    (_highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                 (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
-                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
-                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
+                    ((_highestBidder != address(0) && bidBps < auction.highestBidBps) ||
+                        (_highestBidder == address(0) && bidBps <= auction.startBidBps))),
             "your bid is not the highest"
         );

@@ -177,17 +179,17 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

             if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
-                if (auction.highestBidder != address(0)) {
+                if (_highestBidder != address(0)) {
                     uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
                         MAX_BPS);
-                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
+                    erc20.safeTransfer(_highestBidder, previousBidAmount);
                 }

                 uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                 erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
             } else {
-                if (auction.highestBidder != address(0)) {
-                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
+                if (_highestBidder != address(0)) {
+                    erc20.safeTransfer(_highestBidder, auction.marketDebt[auction.markets[i]]);
                 }

                 erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
@@ -198,7 +200,7 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         auction.highestBidBps = bidBps;
         auction.highestBidBlock = block.number;

-        emit BidPlaced(comptroller, auction.startBlock, bidBps, msg.sender);
+        emit BidPlaced(comptroller, _startBlock, bidBps, msg.sender);
     }

     /**
@@ -276,11 +278,12 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         Auction storage auction = auctions[comptroller];

         require(_isStarted(auction), "no on-going auction");
-        require(_isStale(auction), "you need to wait for more time for first bidder");
+        uint256 _startBlock = auction.startBlock;
+        require(_isStale(_startBlock, auction.highestBidder), "you need to wait for more time for first bidder");

         auction.status = AuctionStatus.ENDED;

-        emit AuctionRestarted(comptroller, auction.startBlock);
+        emit AuctionRestarted(comptroller, _startBlock);
         _startAuction(comptroller);
     }

@@ -462,10 +465,9 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
     /**
      * @dev Checks if the auction is stale, i.e. there's no bidder and the auction
      *   was started more than waitForFirstBidder blocks ago.
-     * @param auction The auction to query the status for
      */
-    function _isStale(Auction storage auction) internal view returns (bool) {
-        bool noBidder = auction.highestBidder == address(0);
-        return noBidder && (block.number > auction.startBlock + waitForFirstBidder);
+    function _isStale(uint256 _startBlock, address _highestBidder) internal view returns (bool) {
+        bool noBidder = _highestBidder == address(0);
+        return noBidder && (block.number > _startBlock + waitForFirstBidder);
     }
 }
```

## Return values from external calls can be cached to avoid unnecessary call
External calls are expensive as they use the `STATICCALL`/`CALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unecessary `STATICCALL`/`CALL`.

Total Instances: `2`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L238-L240

*Gas Savings for `RiskFund.swapPoolsAssets`, obtained via protocol's tests: Avg 7142 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  45850   |  613508  |  366040 |    7   |
| After  |  45850   |  603600  |  358898 |    7   |

### Cache `ComptrollerViewInterface(comptroller).oracle()` to save 1 External Call
```solidity
File: contracts/RiskFund/RiskFund.sol
238:        ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken)); // @audit: 1st external call
239:
240:        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice( // @audit: 2nd external call
```
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..949a117 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -234,10 +234,11 @@ contract RiskFund is

         address underlyingAsset = VTokenInterface(address(vToken)).underlying();
         uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];
+
+        PriceOracle _oracle = ComptrollerViewInterface(comptroller).oracle();
+        _oracle.updatePrice(address(vToken));

-        ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));
-
-        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
+        uint256 underlyingAssetPrice = _oracle.getUnderlyingPrice(
             address(vToken)
         );
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402-L403

### Cache `VToken(vToken).borrowIndex()` outside of loop to save an External Call per loop iteration (100 * num_iterations)
```solidity
File: contracts/Comptroller.sol
402:        for (uint256 i; i < rewardDistributorsCount; ++i) {
403:            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() }); // @audit: external call on each iteration
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..24924f1 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -399,8 +399,8 @@ contract Comptroller is
         // Keep the flywheel moving
         uint256 rewardDistributorsCount = rewardsDistributors.length;

+        Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
         for (uint256 i; i < rewardDistributorsCount; ++i) {
-            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
             rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
         }
```

## A mapping is more efficient than an array
Fetching data from an array is more expensive than fetching data from a mapping. Fetching data from an array will require iterating over the array until ou reach your desired data. When using a mapping you only need to know the key in order to fetch the data from the exact slot it is stored in. [Source](https://twitter.com/pcaversaccio/status/1464523336730480640)

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/ComptrollerStorage.sol#L83

*Gas Savings for `PoolRegistry.addMarket`, obtained via protocol's tests: Avg 248 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1270185   |  1601274  |  1378668 |    58   |
| Before |  1269971   |  1601189  |  1378420 |    58   |

```solidity
File: contracts/ComptrollerStorage.sol
83:    VToken[] public allMarkets;
```
```diff
diff --git a/contracts/ComptrollerStorage.sol b/contracts/ComptrollerStorage.sol
index 31cca94..3c604a6 100644
--- a/contracts/ComptrollerStorage.sol
+++ b/contracts/ComptrollerStorage.sol
@@ -80,7 +80,8 @@ contract ComptrollerStorage {
     mapping(address => Market) public markets;

     /// @notice A list of all markets
-    VToken[] public allMarkets;
+    mapping(uint256 => VToken) public allMarkets;
+    uint256 allMarketsCount;

     /// @notice Borrow caps enforced by borrowAllowed for each vToken address. Defaults to zero which restricts borrowing.
     mapping(address => uint256) public borrowCaps;
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..97a2fe9 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -943,7 +943,7 @@ contract Comptroller is
         rewardsDistributors.push(_rewardsDistributor);
         rewardsDistributorExists[address(_rewardsDistributor)] = true;

-        uint256 marketsCount = allMarkets.length;
+        uint256 marketsCount = allMarketsCount;

         for (uint256 i; i < marketsCount; ++i) {
             _rewardsDistributor.initializeMarket(address(allMarkets[i]));
@@ -1036,7 +1036,16 @@ contract Comptroller is
      * @return markets The list of market addresses
      */
     function getAllMarkets() external view override returns (VToken[] memory) {
-        return allMarkets;
+        uint256 count = allMarketsCount;
+        VToken[] memory markets = new VToken[](count);
+        for (uint256 i; i < count; ) {
+            markets[i] = allMarkets[i];
+            unchecked {
+                ++i;
+            }
+        }
+
+        return markets;
     }

     /**
@@ -1203,15 +1212,16 @@ contract Comptroller is
      * @param vToken The market to support
      */
     function _addMarket(address vToken) internal {
-        uint256 marketsCount = allMarkets.length;
+        uint256 marketsCount = allMarketsCount;

         for (uint256 i; i < marketsCount; ++i) {
             if (allMarkets[i] == VToken(vToken)) {
                 revert MarketAlreadyListed(vToken);
             }
         }
-        allMarkets.push(VToken(vToken));
-        marketsCount = allMarkets.length;
+        allMarkets[marketsCount] = VToken(vToken);
+        marketsCount += 1;
+        allMarketsCount = marketsCount;
         _ensureMaxLoops(marketsCount);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/ComptrollerStorage.sol#L98

*Gas Savings for `VToken.liquidateBorrow`, obtained via protocol's tests: Avg 286 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  216517  |  369742  |  293130 |    4     |
| After  |  216517  |  369170  |  292844 |    4     |

```solidity
File: contracts/ComptrollerStorage.sol
98:    RewardsDistributor[] internal rewardsDistributors;
```
```diff
diff --git a/contracts/ComptrollerStorage.sol b/contracts/ComptrollerStorage.sol
index 31cca94..a9e8049 100644
--- a/contracts/ComptrollerStorage.sol
+++ b/contracts/ComptrollerStorage.sol
@@ -95,7 +95,8 @@ contract ComptrollerStorage {
     mapping(address => mapping(Action => bool)) internal _actionPaused;

     // List of Reward Distributors added
-    RewardsDistributor[] internal rewardsDistributors;
+    mapping(uint256 => RewardsDistributor) internal rewardsDistributors;
+    uint256 rewardsDistributorsCount;

     // Used to check if rewards distributor is added
     mapping(address => bool) internal rewardsDistributorExists;
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..194fe6d 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -269,7 +269,7 @@ contract Comptroller is
         }

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
@@ -299,7 +299,7 @@ contract Comptroller is
         _checkRedeemAllowed(vToken, redeemer, redeemTokens);

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
@@ -371,7 +371,7 @@ contract Comptroller is
         Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
@@ -397,7 +397,7 @@ contract Comptroller is
         }

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
@@ -516,7 +516,7 @@ contract Comptroller is
         }

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].updateRewardTokenSupplyIndex(vTokenCollateral);
@@ -553,7 +553,7 @@ contract Comptroller is
         _checkRedeemAllowed(vToken, src, transferTokens);

         // Keep the flywheel moving
-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
@@ -814,7 +814,7 @@ contract Comptroller is

         _addMarket(address(vToken));

-        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        uint256 rewardDistributorsCount = rewardsDistributorsCount;

         for (uint256 i; i < rewardDistributorsCount; ++i) {
             rewardsDistributors[i].initializeMarket(address(vToken));
@@ -927,7 +927,7 @@ contract Comptroller is
     function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner {
         require(!rewardsDistributorExists[address(_rewardsDistributor)], "already exists");

-        uint256 rewardsDistributorsLength = rewardsDistributors.length;
+        uint256 rewardsDistributorsLength = rewardsDistributorsCount;

         for (uint256 i; i < rewardsDistributorsLength; ++i) {
             address rewardToken = address(rewardsDistributors[i].rewardToken());
@@ -937,10 +937,11 @@ contract Comptroller is
             );
         }

-        uint256 rewardsDistributorsLen = rewardsDistributors.length;
+        uint256 rewardsDistributorsLen = rewardsDistributorsCount;
         _ensureMaxLoops(rewardsDistributorsLen + 1);
-
-        rewardsDistributors.push(_rewardsDistributor);
+
+        rewardsDistributors[rewardsDistributorsLen] = _rewardsDistributor;
+        rewardsDistributorsCount = rewardsDistributorsLen + 1;
         rewardsDistributorExists[address(_rewardsDistributor)] = true;

         uint256 marketsCount = allMarkets.length;
@@ -1117,7 +1118,7 @@ contract Comptroller is
      * @return rewardSpeeds Array of total supply and borrow speeds and reward token for all reward distributors
      */
     function getRewardsByMarket(address vToken) external view returns (RewardSpeeds[] memory rewardSpeeds) {
-        uint256 rewardsDistributorsLength = rewardsDistributors.length;
+        uint256 rewardsDistributorsLength = rewardsDistributorsCount;
         rewardSpeeds = new RewardSpeeds[](rewardsDistributorsLength);
         for (uint256 i; i < rewardsDistributorsLength; ++i) {
             address rewardToken = address(rewardsDistributors[i].rewardToken());
@@ -1142,7 +1143,15 @@ contract Comptroller is
      * @return Array of RewardDistributor addresses
      */
     function getRewardDistributors() public view returns (RewardsDistributor[] memory) {
-        return rewardsDistributors;
+        uint256 count = rewardsDistributorsCount;
+        RewardsDistributor[] memory distributors = new RewardsDistributor[](count);
+        for (uint256 i; i < count; ) {
+            distributors[i] = rewardsDistributors[i];
+            unchecked {
+                ++i;
+            }
+        }
+        return distributors;
     }
```

## Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L804-L810

*Gas Savings for `Comptroller.supportMarket`, obtained via protocol's tests: Avg 89 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  94253   |  108914  |  103153 |    25   |
| After  |  94175   |  108824  |  103064 |    25   |

```solidity
File: contracts/Comptroller.sol
804:        if (markets[address(vToken)].isListed) {
805:            revert MarketAlreadyListed(address(vToken));
806:        }
807:
808:        require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure its really a VToken
809:
810:        Market storage newMarket = markets[address(vToken)];
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..ad41791 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -801,13 +801,13 @@ contract Comptroller is
     function supportMarket(VToken vToken) external {
         _checkSenderIs(poolRegistry);

+        Market storage newMarket = markets[address(vToken)];
         if (markets[address(vToken)].isListed) {
             revert MarketAlreadyListed(address(vToken));
         }

         require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure its really a VToken

-        Market storage newMarket = markets[address(vToken)];
         newMarket.isListed = true;
         newMarket.collateralFactorMantissa = 0;
         newMarket.liquidationThresholdMantissa = 0;
```

## Move calldata pointer to top of for loop to avoid offset calculations
We can avoid unnecessary offset calculations by moving the calldata pointer to the top of the for loop.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L669-L677

*Gas Savings for `Comptroller.liquidateAccount`, obtained via protocol's tests: Avg 194 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  91420   |  373370  |  233098 |    4   |
| After  |  91198   |  373259  |  232904 |    4   |

```solidity
File: contracts/Comptroller.sol
669:        for (uint256 i; i < ordersCount; ++i) {
670:            if (!markets[address(orders[i].vTokenBorrowed)].isListed) {
671:                revert MarketNotListed(address(orders[i].vTokenBorrowed));
672:            }
673:            if (!markets[address(orders[i].vTokenCollateral)].isListed) {
674:                revert MarketNotListed(address(orders[i].vTokenCollateral));
675:            }
676:
677:            LiquidationOrder calldata order = orders[i];
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..2c0abc9 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -667,14 +667,14 @@ contract Comptroller is
         _ensureMaxLoops(ordersCount);

         for (uint256 i; i < ordersCount; ++i) {
-            if (!markets[address(orders[i].vTokenBorrowed)].isListed) {
-                revert MarketNotListed(address(orders[i].vTokenBorrowed));
+            LiquidationOrder calldata order = orders[i];
+            if (!markets[address(order.vTokenBorrowed)].isListed) {
+                revert MarketNotListed(address(order.vTokenBorrowed));
             }
-            if (!markets[address(orders[i].vTokenCollateral)].isListed) {
-                revert MarketNotListed(address(orders[i].vTokenCollateral));
+            if (!markets[address(order.vTokenCollateral)].isListed) {
+                revert MarketNotListed(address(order.vTokenCollateral));
             }

-            LiquidationOrder calldata order = orders[i];
             order.vTokenBorrowed.forceLiquidateBorrow(
                 msg.sender,
                 borrower,
```

## Using storage instead of memory for structs/arrays saves gas
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don't incur extra `Gcoldsload (2100 gas)` for fields that you will never use.

**Note: These are instances that the automated report missed**.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L394

*Gas Savings for `PoolRegistry.createRegistryPool`, obtained via protocol's tests: Avg 853 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  645562  |  683114  |  664419 |    23   |
| After  |  644720  |  682260  |  663566 |    23   |

```solidity
File: contracts/Pool/PoolRegistry.sol
394:        VenusPool memory venusPool = _poolByComptroller[comptroller];
```
```diff
diff --git a/contracts/Pool/PoolRegistry.sol b/contracts/Pool/PoolRegistry.sol
index 5cf376f..7d7c2e6 100644
--- a/contracts/Pool/PoolRegistry.sol
+++ b/contracts/Pool/PoolRegistry.sol
@@ -391,7 +391,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
      * @return The index of the registered Venus pool
      */
     function _registerPool(string calldata name, address comptroller) internal returns (uint256) {
-        VenusPool memory venusPool = _poolByComptroller[comptroller];
+        VenusPool storage venusPool = _poolByComptroller[comptroller];

         require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");
         _ensureValidName(name);
```

## Multiple accesses of a mapping/array should use a storage pointer
Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `stakes[tokenId_]`, save its reference via a storage pointer: `StakeInfo storage stakeInfo = stakes[tokenId_]` and use the pointer instead.

**Note: These are instances the automated report missed**

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L263-L317

*Gas Savings for `PoolRegistry.addMarket`, obtained via protocol's tests: Avg 108 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  1270185   |  1601274  |  1378668 |    58   |
| After  |  1270076   |  1601165  |  1378560 |    58   |

### Cache storage pointer for `_vTokens[input.comptroller]`
```solidity
File: contracts/Pool/PoolRegistry.sol
263:        require(
264:            _vTokens[input.comptroller][input.asset] == address(0),
265:            "PoolRegistry: Market already added for asset comptroller combination"
266:        );
...
317:        _vTokens[input.comptroller][input.asset] = address(vToken);
```
```diff
diff --git a/contracts/Pool/PoolRegistry.sol b/contracts/Pool/PoolRegistry.sol
index 5cf376f..154cccb 100644
--- a/contracts/Pool/PoolRegistry.sol
+++ b/contracts/Pool/PoolRegistry.sol
@@ -260,8 +260,9 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
         require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

         // solhint-disable-next-line reason-string
+        mapping(address => address) storage vTokens_ = _vTokens[input.comptroller];
         require(
-            _vTokens[input.comptroller][input.asset] == address(0),
+            vTokens_[input.asset] == address(0),
             "PoolRegistry: Market already added for asset comptroller combination"
         );

@@ -314,7 +315,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
         comptroller.setMarketSupplyCaps(vTokens, newSupplyCaps);
         comptroller.setMarketBorrowCaps(vTokens, newBorrowCaps);

-        _vTokens[input.comptroller][input.asset] = address(vToken);
+        vTokens_[input.asset] = address(vToken);
         _supportedPools[input.asset].push(input.comptroller);

         IERC20Upgradeable token = IERC20Upgradeable(input.asset);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L72-L75

*Gas Savings for `ProtocolShareReserve.releaseFunds`, obtained via protocol's tests: Avg 73 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  110565   |  194808  |  172312 |    17   |
| After  |  110492   |  194735  |  172239 |    17   |

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol
72:        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
73:
74:        assetsReserves[asset] -= amount;
75:        poolsAssetsReserves[comptroller][asset] -= amount;
```
```diff
diff --git a/contracts/RiskFund/ProtocolShareReserve.sol b/contracts/RiskFund/ProtocolShareReserves.sol
index a669b37..9efaeab 100644
--- a/contracts/RiskFund/ProtocolShareReserve.sol
+++ b/contracts/RiskFund/ProtocolShareReserves.sol
@@ -69,10 +69,11 @@ contract ProtocolShareReserve is Ownable2StepUpgradeable, ExponentialNoError, Re
         uint256 amount
     ) external returns (uint256) {
         require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
-        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
+        mapping(address => uint256) storage _poolsAssetsReserves = poolsAssetsReserves[comptroller];
+        require(amount <= _poolsAssetsReserves[asset], "ProtocolShareReserve: Insufficient pool balance");

         assetsReserves[asset] -= amount;
-        poolsAssetsReserves[comptroller][asset] -= amount;
+        _poolsAssetsReserves[asset] -= amount;
         uint256 protocolIncomeAmount = mul_(
             Exp({ mantissa: amount }),
             div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L335-L336

*Gas Savings for `PoolRegistry.setPoolName`, obtained via protocol's tests: Avg 64 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  47252 |    3   |
| After  |  -   |  -  |  47188 |    3   |

```solidity
File: contracts/Pool/PoolRegistry.sol
335:        string memory oldName = _poolByComptroller[comptroller].name;
336:        _poolByComptroller[comptroller].name = name;
```
```diff
diff --git a/contracts/Pool/PoolRegistry.sol b/contracts/Pool/PoolRegistry.sol
index 5cf376f..b068581 100644
--- a/contracts/Pool/PoolRegistry.sol
+++ b/contracts/Pool/PoolRegistry.sol
@@ -332,8 +332,9 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
     function setPoolName(address comptroller, string calldata name) external {
         _checkAccessAllowed("setPoolName(address,string)");
         _ensureValidName(name);
-        string memory oldName = _poolByComptroller[comptroller].name;
-        _poolByComptroller[comptroller].name = name;
+        VenusPool storage _pool = _poolByComptroller[comptroller];
+        string memory oldName = _pool.name;
+        _pool.name = name;
         emit PoolNameSet(comptroller, oldName, name);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L628-L631

*Gas Savings for `VToken.increaseAllowance`, obtained via protocol's tests: Avg 76 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  36636 |    1   |
| After  |  -   |  -  |  36560 |    1   |

### Cache storage pointer for `transferAllowances[src]`
```solidity
File: contracts/VToken.sol
628:        address src = msg.sender;
629:        uint256 newAllowance = transferAllowances[src][spender];
630:        newAllowance += addedValue;
631:        transferAllowances[src][spender] = newAllowance;
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..cd968ee 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -625,12 +625,12 @@ contract VToken is
     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
         require(spender != address(0), "invalid spender address");

-        address src = msg.sender;
-        uint256 newAllowance = transferAllowances[src][spender];
+        mapping(address => uint256) storage _transferAllowance = transferAllowances[msg.sender];
+        uint256 newAllowance = _transferAllowance[spender];
         newAllowance += addedValue;
-        transferAllowances[src][spender] = newAllowance;
+        _transferAllowance[spender] = newAllowance;

-        emit Approval(src, spender, newAllowance);
+        emit Approval(msg.sender, spender, newAllowance);
         return true;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L649-L655

*Gas Savings for `VToken.decreaseAllowance`, obtained via protocol's tests: Avg 70 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  36685 |    1   |
| After  |  -   |  -  |  36615 |    1   |

```solidity
File: contracts/VToken.sol
649:        uint256 currentAllowance = transferAllowances[src][spender];
650:        require(currentAllowance >= subtractedValue, "decreased allowance below zero");
651:        unchecked {
652:            currentAllowance -= subtractedValue;
653:        }
654:
655:        transferAllowances[src][spender] = currentAllowance;
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..4e3ca4d 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -644,17 +644,17 @@ contract VToken is
      */
     function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
         require(spender != address(0), "invalid spender address");
-
-        address src = msg.sender;
-        uint256 currentAllowance = transferAllowances[src][spender];
+
+        mapping(address => uint256) storage _transferAllowance = transferAllowances[msg.sender];
+        uint256 currentAllowance = _transferAllowance[spender];
         require(currentAllowance >= subtractedValue, "decreased allowance below zero");
         unchecked {
             currentAllowance -= subtractedValue;
         }

-        transferAllowances[src][spender] = currentAllowance;
+        _transferAllowance[spender] = currentAllowance;

-        emit Approval(src, spender, currentAllowance);
+        emit Approval(msg.sender, spender, currentAllowance);
         return true;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L236-L250

*Gas Savings for `RiskFund.swapPoolsAssets`, obtained via protocol's tests: Avg 60 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  45850   |  613508  |  366040 |    7   |
| After  |  45850   |  613259  |  365980 |    7   |

```solidity
File: contracts/RiskFund/RiskFund.sol
236:        uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];
...
248:            if (amountInUsd >= minAmountToConvert) {
249:                assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
250:                poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
```
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..c8827bd 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -233,7 +233,8 @@ contract RiskFund is
         uint256 totalAmount;

         address underlyingAsset = VTokenInterface(address(vToken)).underlying();
-        uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];
+        mapping(address => uint256) storage _poolsAssetsReserves = poolsAssetsReserves[comptroller];
+        uint256 balanceOfUnderlyingAsset = _poolsAssetsReserves[underlyingAsset];

         ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));

@@ -247,7 +248,7 @@ contract RiskFund is

             if (amountInUsd >= minAmountToConvert) {
                 assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
-                poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
+                _poolsAssetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;

                 if (underlyingAsset != convertibleBaseAsset) {
                     require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L424-L425

### Cache storage pointer for `accountBorrows[borrower]`

**Note: This function, `healBorrow`, is not benchmarked in tests so the savings are ommitted for this instance**.
```solidity
File: contracts/VToken.sol
424:        accountBorrows[borrower].principal = 0;
425:        accountBorrows[borrower].interestIndex = borrowIndex;
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..20d05a5 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -420,9 +420,10 @@ contract VToken is
             emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);
             emit BadDebtIncreased(borrower, badDebtDelta, badDebtOld, badDebtNew);
         }
-
-        accountBorrows[borrower].principal = 0;
-        accountBorrows[borrower].interestIndex = borrowIndex;
+
+        BorrowSnapshot storage _accountBorrow = accountBorrows[borrower];
+        _accountBorrow.principal = 0;
+        _accountBorrow.interestIndex = borrowIndex;
         totalBorrows = totalBorrowsNew;

         emit HealBorrow(payer, borrower, repayAmount);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L908-L909

### Cache storage pointer for `accountBorrows[borrower]`
```solidity
File: contracts/VToken.sol
908:        accountBorrows[borrower].principal = accountBorrowsNew;
909:        accountBorrows[borrower].interestIndex = borrowIndex;
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..20efff2 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -905,8 +905,9 @@ contract VToken is
          * We write the previously calculated values into storage.
          *  Note: Avoid token reentrancy attacks by writing increased borrow before external transfer.
         `*/
-        accountBorrows[borrower].principal = accountBorrowsNew;
-        accountBorrows[borrower].interestIndex = borrowIndex;
+        BorrowSnapshot storage _accountBorrow = accountBorrows[borrower];
+        _accountBorrow.principal = accountBorrowsNew;
+        _accountBorrow.interestIndex = borrowIndex;
         totalBorrows = totalBorrowsNew;

         /*
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L969-L970

```solidity
File: contracts/VToken.sol
969:        accountBorrows[borrower].principal = accountBorrowsNew;
970:        accountBorrows[borrower].interestIndex = borrowIndex;
```
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..76771a4 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -966,8 +966,9 @@ contract VToken is
         uint256 totalBorrowsNew = totalBorrows - actualRepayAmount;

         /* We write the previously calculated values into storage */
-        accountBorrows[borrower].principal = accountBorrowsNew;
-        accountBorrows[borrower].interestIndex = borrowIndex;
+        BorrowSnapshot storage _accountBorrow = accountBorrows[borrower];
+        _accountBorrow.principal = accountBorrowsNew;
+        _accountBorrow.interestIndex = borrowIndex;
         totalBorrows = totalBorrowsNew;

         /* We emit a RepayBorrow event */
```

## Use do while loops instead of for loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L166

*Gas Savings for `RiskFund.swapPoolsAssets`, obtained via protocol's tests: Avg 73 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  45850   |  613508  |  366040 |    7     |
| After  |  45838   |  613399  |  365967 |    7     |

```solidity
File: contracts/RiskFund/RiskFund.sol
166:        for (uint256 i; i < marketsCount; ++i) {
```
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..b6fdc0c 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -162,18 +162,22 @@ contract RiskFund is
         uint256 marketsCount = markets.length;

         _ensureMaxLoops(marketsCount);
-
-        for (uint256 i; i < marketsCount; ++i) {
-            VToken vToken = VToken(markets[i]);
-            address comptroller = address(vToken.comptroller());
-
-            PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
-            require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
-            require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");
-
-            uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
-            poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;
-            totalAmount = totalAmount + swappedTokens;
+
+        if (marketsCount != 0) {
+            uint256 i;
+            do {
+                VToken vToken = VToken(markets[i]);
+                address comptroller = address(vToken.comptroller());
+
+                PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
+                require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
+                require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");
+
+                uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
+                poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;
+                totalAmount = totalAmount + swappedTokens;
+                ++i;
+            } while(i < marketsCount);
         }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L282

*Gas Savings for `RewardsDistributor.claimRewardToken`, obtained via protocol's tests: Avg 61 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  262655  |  291169  |  276621 |    3     |
| After  |  262594  |  291108  |  276560 |    3     |

```solidity
File: contracts/Rewards/RewardsDistributor.sol
282:        for (uint256 i; i < vTokensCount; ++i) {
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..90e8a28 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -278,8 +278,9 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
         uint256 vTokensCount = vTokens.length;

         _ensureMaxLoops(vTokensCount);
-
-        for (uint256 i; i < vTokensCount; ++i) {
+
+        uint256 i;
+        do {
             VToken vToken = vTokens[i];
             require(comptroller.isMarketListed(vToken), "market must be listed");
             Exp memory borrowIndex = Exp({ mantissa: vToken.borrowIndex() });
@@ -287,7 +288,8 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
             _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
             _updateRewardTokenSupplyIndex(address(vToken));
             _distributeSupplierRewardToken(address(vToken), holder);
-        }
+            ++i;
+        } while(i < vTokensCount);
         rewardTokenAccrued[holder] = _grantRewardToken(holder, rewardTokenAccrued[holder]);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L209

*Gas Savings for `RewardsDistributor.setRewardTokenSpeeds`, obtained via protocol's tests: Avg 53 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -   |  -  |  208377 |    20   |
| After  |  -   |  -  |  208324 |    20   |

```solidity
File: contracts/Rewards/RewardsDistributor.sol
209:        for (uint256 i; i < numTokens; ++i) {
```
```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..52d9cbc 100644
--- a/contracts/Rewards/RewardsDistributor.sol
+++ b/contracts/Rewards/RewardsDistributor.sol
@@ -205,10 +205,12 @@ contract RewardsDistributor is ExponentialNoError, Ownable2StepUpgradeable, Acce
             numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,
             "RewardsDistributor::setRewardTokenSpeeds invalid input"
         );
-
-        for (uint256 i; i < numTokens; ++i) {
+
+        uint256 i;
+        do {
             _setRewardTokenSpeed(vTokens[i], supplySpeeds[i], borrowSpeeds[i]);
-        }
+            ++i;
+        } while(i < numTokens);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L389

*Gas Savings for `Shortfall.startAuction`, obtained via protocol's tests: Avg 57 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  284934  |  285372  |  285153 |    6     |
| After  |  284877  |  285315  |  285096 |    6     |

```solidity
File: contracts/Shortfall/Shortfall.sol
389:        for (uint256 i; i < marketsCount; ++i) {
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..0917864 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -385,17 +385,20 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

         uint256[] memory marketsDebt = new uint256[](marketsCount);
         auction.markets = new VToken[](marketsCount);
-
-        for (uint256 i; i < marketsCount; ++i) {
-            uint256 marketBadDebt = vTokens[i].badDebt();
-
-            priceOracle.updatePrice(address(vTokens[i]));
-            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
-
-            poolBadDebt = poolBadDebt + usdValue;
-            auction.markets[i] = vTokens[i];
-            auction.marketDebt[vTokens[i]] = marketBadDebt;
-            marketsDebt[i] = marketBadDebt;
+        {
+            uint256 i;
+            do {
+                uint256 marketBadDebt = vTokens[i].badDebt();
+
+                priceOracle.updatePrice(address(vTokens[i]));
+                uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
+
+                poolBadDebt = poolBadDebt + usdValue;
+                auction.markets[i] = vTokens[i];
+                auction.marketDebt[vTokens[i]] = marketBadDebt;
+                marketsDebt[i] = marketBadDebt;
+                ++i;
+            } while(i < marketsCount);
         }

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L223

*Gas Savings for `Shortfall.closeAuction`, obtained via protocol's tests: Avg 55 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  189929  |  193944  |  191937 |    4     |
| After  |  189874  |  193889  |  191882 |    4     |

```solidity
File: contracts/Shortfall/Shortfall.sol
223:        for (uint256 i; i < marketsCount; ++i) {
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..5a1ee4e 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -219,8 +219,9 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         uint256[] memory marketsDebt = new uint256[](marketsCount);

         auction.status = AuctionStatus.ENDED;
-
-        for (uint256 i; i < marketsCount; ++i) {
+
+        uint256 i;
+        do {
             VToken vToken = VToken(address(auction.markets[i]));
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

@@ -234,7 +235,8 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
             }

             auction.markets[i].badDebtRecovered(marketsDebt[i]);
-        }
+            ++i;
+        } while(i < marketsCount);

         uint256 riskFundBidAmount;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L175

*Gas Savings for `Shortfall.placeBid`, obtained via protocol's tests: Avg 61 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  140790  |  216414  |  185429 |    5     |
| After  |  140729  |  216353  |  185368 |    5     |

```solidity
File: contracts/Shortfall/Shortfall.sol
175:        for (uint256 i; i < marketsCount; ++i) {
```
```diff
diff --git a/contracts/Shortfall/Shortfall.sol b/contracts/Shortfall/Shortfall.sol
index 6e6beb8..e143611 100644
--- a/contracts/Shortfall/Shortfall.sol
+++ b/contracts/Shortfall/Shortfall.sol
@@ -172,7 +172,8 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua
         );

         uint256 marketsCount = auction.markets.length;
-        for (uint256 i; i < marketsCount; ++i) {
+        uint256 i;
+        do {
             VToken vToken = VToken(address(auction.markets[i]));
             IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

@@ -192,7 +193,8 @@ contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGua

                 erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
             }
-        }
+            ++i;
+        } while(i < marketsCount);

         auction.highestBidder = msg.sender;
         auction.highestBidBps = bidBps;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L162

*Gas Savings for `Comptroller.enterMarkets`, obtained via protocol's tests: Avg 55 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  41896   |  209652  |  132444 |    110   |
| After  |  41855   |  209583  |  132389 |    110   |

```solidity
File: contracts/Comptroller.sol
162:        for (uint256 i; i < len; ++i) {
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..d9ffffe 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -159,12 +159,14 @@ contract Comptroller is
         _ensureMaxLoops(accountAssetsLen + len);

         uint256[] memory results = new uint256[](len);
-        for (uint256 i; i < len; ++i) {
+        uint256 i;
+        do {
             VToken vToken = VToken(vTokens[i]);

             _addToMarket(vToken, msg.sender);
             results[i] = NO_ERROR;
-        }
+            ++i;
+        } while(i < len);

         return results;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L611

*Gas Savings for `Comptroller.healAccount`, obtained via protocol's tests: Avg 68 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  84584   |  331091  |  223131 |    10   |
| After  |  84506   |  331030  |  223063 |    10   |

```solidity
File: contracts/Comptroller.sol
611:        for (uint256 i; i < userAssetsCount; ++i) {
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..858f6bb 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -607,8 +607,9 @@ contract Comptroller is
         if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
             revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
         }
-
-        for (uint256 i; i < userAssetsCount; ++i) {
+
+        uint256 i;
+        do {
             VToken market = userAssets[i];

             (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);
@@ -622,7 +623,8 @@ contract Comptroller is
             if (borrowBalance != 0) {
                 market.healBorrow(liquidator, user, repaymentAmount);
             }
-        }
+            ++i;
+        } while(i < userAssetsCount);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L690

*Gas Savings for `Comptroller.liquidateAccount`, obtained via protocol's tests: Avg 62 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  91420   |  373370  |  233098 |    4   |
| After  |  91351   |  373315  |  233036 |    4   |

```solidity
File: contracts/Comptroller.sol
690:        for (uint256 i; i < marketsCount; ++i) {
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..a28f88b 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -686,11 +686,13 @@ contract Comptroller is

         VToken[] memory borrowMarkets = accountAssets[borrower];
         uint256 marketsCount = borrowMarkets.length;
-
-        for (uint256 i; i < marketsCount; ++i) {
+
+        uint256 i;
+        do {
             (, uint256 borrowBalance, ) = _safeGetAccountSnapshot(borrowMarkets[i], borrower);
             require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
-        }
+            ++i;
+        } while(i < marketsCount);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1307

*Gas Savings for `Comptroller.liquidateAccount`, obtained via protocol's tests: Avg 53 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  91420   |  373370  |  233098 |    4   |
| After  |  91353   |  373332  |  233045 |    4   |

```solidity
File: contracts/Comptroller.sol
1307:        for (uint256 i; i < assetsCount; ++i) {
```
```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..55f1741 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -1303,47 +1303,50 @@ contract Comptroller is
         // For each asset the account is in
         VToken[] memory assets = accountAssets[account];
         uint256 assetsCount = assets.length;
-
-        for (uint256 i; i < assetsCount; ++i) {
-            VToken asset = assets[i];
-
-            // Read the balances and exchange rate from the vToken
-            (uint256 vTokenBalance, uint256 borrowBalance, uint256 exchangeRateMantissa) = _safeGetAccountSnapshot(
-                asset,
-                account
-            );
-
-            // Get the normalized price of the asset
-            Exp memory oraclePrice = Exp({ mantissa: _safeGetUnderlyingPrice(asset) });
-
-            // Pre-compute conversion factors from vTokens -> usd
-            Exp memory vTokenPrice = mul_(Exp({ mantissa: exchangeRateMantissa }), oraclePrice);
-            Exp memory weightedVTokenPrice = mul_(weight(asset), vTokenPrice);
-
-            // weightedCollateral += weightedVTokenPrice * vTokenBalance
-            snapshot.weightedCollateral = mul_ScalarTruncateAddUInt(
-                weightedVTokenPrice,
-                vTokenBalance,
-                snapshot.weightedCollateral
-            );
-
-            // totalCollateral += vTokenPrice * vTokenBalance
-            snapshot.totalCollateral = mul_ScalarTruncateAddUInt(vTokenPrice, vTokenBalance, snapshot.totalCollateral);
-
-            // borrows += oraclePrice * borrowBalance
-            snapshot.borrows = mul_ScalarTruncateAddUInt(oraclePrice, borrowBalance, snapshot.borrows);
-
-            // Calculate effects of interacting with vTokenModify
-            if (asset == vTokenModify) {
-                // redeem effect
-                // effects += tokensToDenom * redeemTokens
-                snapshot.effects = mul_ScalarTruncateAddUInt(weightedVTokenPrice, redeemTokens, snapshot.effects);
-
-                // borrow effect
-                // effects += oraclePrice * borrowAmount
-                snapshot.effects = mul_ScalarTruncateAddUInt(oraclePrice, borrowAmount, snapshot.effects);
-            }
-        }
+        if (assetsCount > 0) {
+            uint256 i;
+            do {
+                VToken asset = assets[i];
+
+                // Read the balances and exchange rate from the vToken
+                (uint256 vTokenBalance, uint256 borrowBalance, uint256 exchangeRateMantissa) = _safeGetAccountSnapshot(
+                    asset,
+                    account
+                );
+
+                // Get the normalized price of the asset
+                Exp memory oraclePrice = Exp({ mantissa: _safeGetUnderlyingPrice(asset) });
+
+                // Pre-compute conversion factors from vTokens -> usd
+                Exp memory vTokenPrice = mul_(Exp({ mantissa: exchangeRateMantissa }), oraclePrice);
+                Exp memory weightedVTokenPrice = mul_(weight(asset), vTokenPrice);
+
+                // weightedCollateral += weightedVTokenPrice * vTokenBalance
+                snapshot.weightedCollateral = mul_ScalarTruncateAddUInt(
+                    weightedVTokenPrice,
+                    vTokenBalance,
+                    snapshot.weightedCollateral
+                );
+
+                // totalCollateral += vTokenPrice * vTokenBalance
+                snapshot.totalCollateral = mul_ScalarTruncateAddUInt(vTokenPrice, vTokenBalance, snapshot.totalCollateral);
+
+                // borrows += oraclePrice * borrowBalance
+                snapshot.borrows = mul_ScalarTruncateAddUInt(oraclePrice, borrowBalance, snapshot.borrows);
+
+                // Calculate effects of interacting with vTokenModify
+                if (asset == vTokenModify) {
+                    // redeem effect
+                    // effects += tokensToDenom * redeemTokens
+                    snapshot.effects = mul_ScalarTruncateAddUInt(weightedVTokenPrice, redeemTokens, snapshot.effects);
+
+                    // borrow effect
+                    // effects += oraclePrice * borrowAmount
+                    snapshot.effects = mul_ScalarTruncateAddUInt(oraclePrice, borrowAmount, snapshot.effects);
+                }
+                ++i;
+            } while(i < assetsCount);
+        }
```

## Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (`scratch space` + `free memory pointer` + `zero slot`), which can potentially allow us to avoid memory expansion costs.

**Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if necessary**.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L239-L245

*Gas Savings for `PoolRegistry.createRegistryPool`, obtained via protocol's tests: Avg 1049 gas*

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  645562   |  683114   |  664419  |   23   | 
| After  |  644525   |  682065   |  663370  |   23   | 

```solidity
File: contracts/Pool/PoolRegistry.sol
239:        comptrollerProxy.setCloseFactor(closeFactor);
240:        comptrollerProxy.setLiquidationIncentive(liquidationIncentive);
241:        comptrollerProxy.setMinLiquidatableCollateral(minLiquidatableCollateral);
242:        comptrollerProxy.setPriceOracle(PriceOracle(priceOracle));
243:
244:        // Start transferring ownership to msg.sender
245:        comptrollerProxy.transferOwnership(msg.sender);
```
```diff
diff --git a/contracts/Pool/PoolRegistry.sol b/contracts/Pool/PoolRegistry.sol
index 5cf376f..b2d696a 100644
--- a/contracts/Pool/PoolRegistry.sol
+++ b/contracts/Pool/PoolRegistry.sol
@@ -236,13 +236,28 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
         uint256 poolId = _registerPool(name, proxyAddress);

         // Set Venus pool parameters
-        comptrollerProxy.setCloseFactor(closeFactor);
-        comptrollerProxy.setLiquidationIncentive(liquidationIncentive);
-        comptrollerProxy.setMinLiquidatableCollateral(minLiquidatableCollateral);
-        comptrollerProxy.setPriceOracle(PriceOracle(priceOracle));
-
-        // Start transferring ownership to msg.sender
-        comptrollerProxy.transferOwnership(msg.sender);
+        assembly {
+            // function signature for setCloseFactor(uint256)
+            mstore(0x00, 0x12348e96)
+            mstore(0x20, calldataload(0x44))
+            if iszero(call(gas(), comptrollerProxy, 0x00, 0x1c, 0x24, 0x00, 0x00)) {revert(0, 0)}
+            // function signature for setLiquidationIncentive(uint256)
+            mstore(0x00, 0xa8431081)
+            mstore(0x20, calldataload(0x64))
+            if iszero(call(gas(), comptrollerProxy, 0x00, 0x1c, 0x24, 0x00, 0x00)) {revert(0, 0)}
+            // function signature for setMinLiquidatableCollateral(uint256)
+            mstore(0x00, 0x520b6c74)
+            mstore(0x20, calldataload(0x84))
+            if iszero(call(gas(), comptrollerProxy, 0x00, 0x1c, 0x24, 0x00, 0x00)) {revert(0, 0)}
+            // function signature for setPriceOracle(address)
+            mstore(0x00, 0x530e784f)
+            mstore(0x20, calldataload(0xa4))
+            if iszero(call(gas(), comptrollerProxy, 0x00, 0x1c, 0x24, 0x00, 0x00)) {revert(0, 0)}
+            // function signature for transferOwnership(address)
+            mstore(0x00, 0xf2fde38b)
+            mstore(0x20, caller())
+            if iszero(call(gas(), comptrollerProxy, 0x00, 0x1c, 0x24, 0x00, 0x00)) {revert(0, 0)}
+        }

         // Register the pool with this PoolRegistry
         return (poolId, proxyAddress);
```

## GasReport output with all optimizations applied
```js
·---------------------------------------------------------------------------------------------|---------------------------|-------------|-----------------------------·
|                                    Solc version: 0.8.13                                     ·  Optimizer enabled: true  ·  Runs: 200  ·  Block limit: 30000000 gas  │
······························································································|···························|·············|······························
|  Methods                                                                                                                                                            │
························································|·····································|·············|·············|·············|···············|··············
|  Contract                                             ·  Method                             ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
························································|·····································|·············|·············|·············|···············|··············
|  @openzeppelin/contracts/token/ERC20/ERC20.sol:ERC20  ·  approve                            ·      53542  ·      56324  ·      54933  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  @openzeppelin/contracts/token/ERC20/ERC20.sol:ERC20  ·  decreaseAllowance                  ·          -  ·          -  ·      36615  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  @openzeppelin/contracts/token/ERC20/ERC20.sol:ERC20  ·  increaseAllowance                  ·          -  ·          -  ·      36560  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  @openzeppelin/contracts/token/ERC20/ERC20.sol:ERC20  ·  transfer                           ·      66757  ·     223910  ·     145334  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  @openzeppelin/contracts/token/ERC20/ERC20.sol:ERC20  ·  transferFrom                       ·      50770  ·     224897  ·      92435  ·            5  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  AccessControlManager                                 ·  giveCallPermission                 ·      54635  ·      56399  ·      55182  ·           56  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  AccessControlManager                                 ·  revokeCallPermission               ·      32989  ·      33604  ·      33344  ·            5  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Beacon                                               ·  upgradeTo                          ·          -  ·          -  ·      32741  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  acceptOwnership                    ·          -  ·          -  ·      38453  ·           22  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  addRewardsDistributor              ·     103680  ·     231643  ·     174683  ·           38  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  enterMarkets                       ·      41743  ·     209386  ·     132238  ·          110  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  exitMarket                         ·      39603  ·      82114  ·      67959  ·            7  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  healAccount                        ·      82328  ·     322502  ·     218001  ·           10  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  liquidateAccount                   ·      83215  ·     362507  ·     223536  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  preBorrowHook                      ·      57280  ·     122228  ·     100579  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setAccessControlManager            ·          -  ·          -  ·      35061  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setActionsPaused                   ·      38467  ·     344345  ·     107736  ·            7  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setCollateralFactor                ·          -  ·          -  ·      49373  ·            9  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setLiquidationIncentive            ·      40851  ·      57951  ·      47074  ·           11  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setMarketBorrowCaps                ·      61812  ·     138362  ·     133859  ·           17  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setMarketSupplyCaps                ·      55971  ·      62477  ·      61853  ·           12  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setMaxLoopsLimit                   ·          -  ·          -  ·      37511  ·            8  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setMinLiquidatableCollateral       ·      40966  ·      40978  ·      40970  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  setPriceOracle                     ·      37740  ·      54884  ·      42676  ·           33  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Comptroller                                          ·  supportMarket                      ·      89413  ·     104191  ·      98338  ·           25  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ComptrollerHarness                                   ·  harnessFastForward                 ·      33906  ·      51006  ·      40269  ·           43  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ERC20Harness                                         ·  approve                            ·      26290  ·      46202  ·      40908  ·           70  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ERC20Harness                                         ·  harnessSetBalance                  ·      22230  ·      51916  ·      38609  ·          194  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ERC20Harness                                         ·  harnessSetFailTransferFromAddress  ·      24297  ·      44221  ·      25226  ·           65  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ERC20Harness                                         ·  harnessSetFailTransferToAddress    ·      24274  ·      51629  ·      28454  ·           41  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  FeeToken                                             ·  allocateTo                         ·          -  ·          -  ·      51215  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  FeeToken                                             ·  approve                            ·          -  ·          -  ·      46189  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  HarnessMaxLoopsLimitHelper                           ·  setMaxLoopsLimit                   ·          -  ·          -  ·      45030  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  MockPriceOracle                                      ·  setPrice                           ·      24259  ·      49200  ·      44935  ·           41  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  MockToken                                            ·  approve                            ·      26392  ·      46292  ·      46117  ·          145  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  MockToken                                            ·  faucet                             ·      33523  ·      67795  ·      55331  ·          161  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  MockToken                                            ·  transfer                           ·      46666  ·      51490  ·      48135  ·           23  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  addMarket                          ·    1225743  ·    1549309  ·    1334768  ·           58  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  createRegistryPool                 ·     600109  ·     637649  ·     618956  ·           23  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  setPoolName                        ·          -  ·          -  ·      47241  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  setProtocolShareReserve            ·      37924  ·      40756  ·      40476  ·           20  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  setShortfallContract               ·      37945  ·      40748  ·      40472  ·           20  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  PoolRegistry                                         ·  updatePoolMetadata                 ·     114314  ·     114362  ·     114348  ·            5  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ProtocolShareReserve                                 ·  releaseFunds                       ·     110383  ·     194480  ·     171992  ·           17  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ProtocolShareReserve                                 ·  setPoolRegistry                    ·      35002  ·      55038  ·      47145  ·           12  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  ProtocolShareReserve                                 ·  updateAssetsState                  ·          -  ·          -  ·      89816  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RewardsDistributor                                   ·  claimRewardToken                   ·     262553  ·     291067  ·     276519  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RewardsDistributor                                   ·  setContributorRewardTokenSpeed     ·          -  ·          -  ·      77792  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RewardsDistributor                                   ·  setRewardTokenSpeeds               ·          -  ·          -  ·     207399  ·           20  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RewardsDistributor                                   ·  updateContributorRewards           ·          -  ·          -  ·      60671  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RiskFund                                             ·  setMinAmountToConvert              ·          -  ·          -  ·      43969  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RiskFund                                             ·  setPancakeSwapRouter               ·          -  ·          -  ·      37711  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RiskFund                                             ·  setShortfallContractAddress        ·      40240  ·      69837  ·      52201  ·           10  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RiskFund                                             ·  swapPoolsAssets                    ·      45838  ·     601942  ·     357571  ·            7  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  RiskFund                                             ·  transferReserveForAuction          ·          -  ·          -  ·      74187  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  closeAuction                       ·     186297  ·     190916  ·     188607  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  placeBid                           ·     137896  ·     214827  ·     183262  ·            5  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  restartAuction                     ·          -  ·          -  ·     114056  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  startAuction                       ·     284405  ·     284752  ·     284579  ·            6  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  updateIncentiveBps                 ·          -  ·          -  ·      43904  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  updateMinimumPoolBadDebt           ·          -  ·          -  ·      43919  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  updateNextBidderBlockLimit         ·          -  ·          -  ·      43997  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  updatePoolRegistry                 ·      21432  ·      55038  ·      45331  ·            7  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  Shortfall                                            ·  updateWaitForFirstBidder           ·          -  ·          -  ·      43995  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  accrueInterest                     ·      33031  ·      69644  ·      47522  ·            7  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  addReserves                        ·      97803  ·     107403  ·     106838  ·           17  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  borrow                             ·     182388  ·     362989  ·     319410  ·           31  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  liquidateBorrow                    ·     216418  ·     355088  ·     285753  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  mint                               ·      99469  ·     262324  ·     153899  ·           71  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  redeem                             ·     124819  ·     268761  ·     182396  ·            5  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  redeemUnderlying                   ·          -  ·          -  ·     124752  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  reduceReserves                     ·     156023  ·     192360  ·     184375  ·           18  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  repayBorrow                        ·     101552  ·     171782  ·     136740  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  repayBorrowBehalf                  ·          -  ·          -  ·     102043  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  seize                              ·          -  ·          -  ·     100312  ·            4  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  setInterestRateModel               ·          -  ·          -  ·      46118  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  setProtocolSeizeShare              ·          -  ·          -  ·      45923  ·            2  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  setReserveFactor                   ·          -  ·          -  ·      47960  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VToken                                               ·  sweepToken                         ·          -  ·          -  ·      67484  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessBorrowFresh                 ·          -  ·          -  ·     148872  ·            6  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessExchangeRateDetails         ·      33758  ·      76146  ·      45007  ·           17  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessLiquidateBorrowFresh        ·          -  ·          -  ·     170387  ·            6  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessMintFresh                   ·          -  ·          -  ·     128329  ·            6  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessRedeemFresh                 ·      87399  ·      87545  ·      87472  ·            8  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessRepayBorrowFresh            ·      83659  ·      83671  ·      83665  ·            8  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetAccountBorrows           ·      34046  ·      74254  ·      58685  ·           43  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetAccrualBlockNumber       ·          -  ·          -  ·      50924  ·           39  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetBlockNumber              ·      51013  ·      51277  ·      51029  ·           39  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetBorrowIndex              ·      31148  ·      34260  ·      31434  ·           40  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetExchangeRate             ·      31212  ·      73108  ·      66369  ·           47  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetReserveFactorFresh       ·      39597  ·      39621  ·      39613  ·            3  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetTotalBorrows             ·      31009  ·      51293  ·      43307  ·           46  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetTotalReserves            ·          -  ·          -  ·      51047  ·            1  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
|  VTokenHarness                                        ·  harnessSetTotalSupply              ·      31160  ·      51060  ·      48315  ·           29  ·          -  │
························································|·····································|·············|·············|·············|···············|··············
```