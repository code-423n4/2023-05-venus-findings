### Table Of Contents
- [FINDINGS](#findings)
- [Massive 15k per tx gas savings - use 1 and 2 for Reentrancy guard](#massive-15k-per-tx-gas-savings---use-1-and-2-for-reentrancy-guard)
- [The result of a function call should be cached rather than re-calling the function](#the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function)
  - [VToken.sol.sweepToken(): The results of owner() can be cached rather than call it twice](#vtokensolsweeptoken-the-results-of-owner-can-be-cached-rather-than-call-it-twice)
  - [RiskFund.sol.\_swapAsset(): Results of ComptrollerViewInterface(comptroller).oracle() should be cached](#riskfundsol_swapasset-results-of-comptrollerviewinterfacecomptrolleroracle-should-be-cached)
- [Looping on a storage variable](#looping-on-a-storage-variable)
  - [PoolRegistry.sol.getAllPools(): \_numberOfPools is a storage variable](#poolregistrysolgetallpools-_numberofpools-is-a-storage-variable)
- [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
  - [RewardsDistributor.sol.updateContributorRewards(): Emit `contributorAccrued` instead of `rewardTokenAccrued[contributor]`](#rewardsdistributorsolupdatecontributorrewards-emit-contributoraccrued-instead-of-rewardtokenaccruedcontributor)
  - [Instead of reading from storage we can do the arithmetic inside the emit block](#instead-of-reading-from-storage-we-can-do-the-arithmetic-inside-the-emit-block)
  - [MaxLoopsLimitHelper.sol.\_setMaxLoopsLimit(): Emit `limit` instead of `maxLoopsLimit`](#maxloopslimithelpersol_setmaxloopslimit-emit-limit-instead-of-maxloopslimit)
- [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
- [Use the cached value if it exists](#use-the-cached-value-if-it-exists)
  - [Avoid reading from storage if a global variable would be equal to the variable being read](#avoid-reading-from-storage-if-a-global-variable-would-be-equal-to-the-variable-being-read)
  - [VToken.sol.\_initialize(): initialExchangeRateMantissa has been cached and the cached value should be used in the require statement](#vtokensol_initialize-initialexchangeratemantissa-has-been-cached-and-the-cached-value-should-be-used-in-the-require-statement)
  - [Use the local variable `underlying_` instead of `underlying`](#use-the-local-variable-underlying_-instead-of-underlying)
- [IF's/require() statements that check input arguments should be at the top of the function](#ifsrequire-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [Comptroller.sol.setCollateralFactor(): Cheaper to check function params before reading from storage](#comptrollersolsetcollateralfactor-cheaper-to-check-function-params-before-reading-from-storage)
  - [VToken.sol: Do all checks for function parameters first before making external call](#vtokensol-do-all-checks-for-function-parameters-first-before-making-external-call)
  - [VToken.sol.\_seize(): Make function parameter checks first](#vtokensol_seize-make-function-parameter-checks-first)
  - [VToken.sol.\_reduceReservesFresh(): Reorder the checks to have cheaper check first](#vtokensol_reducereservesfresh-reorder-the-checks-to-have-cheaper-check-first)
  - [VToken.sol.\_transferTokens(): Cheaper to check function params  than make external calls](#vtokensol_transfertokens-cheaper-to-check-function-params--than-make-external-calls)
  - [Move checks for function arguments above the one reading storage](#move-checks-for-function-arguments-above-the-one-reading-storage)
- [Use calldata instead of memory for function parameters](#use-calldata-instead-of-memory-for-function-parameters)
  - [Use calldata for `name_,symbol_ and riskManagement`](#use-calldata-for-name_symbol_-and-riskmanagement)
- [Duplicated require()/revert() checks should be refactored to a modifier or function](#duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [Nested if is cheaper than single statement using \&\&](#nested-if-is-cheaper-than-single-statement-using-)
- [Don't cache global variables such as msg.sender](#dont-cache-global-variables-such-as-msgsender)
  - [We don't need to cache msg.sender, read it directly](#we-dont-need-to-cache-msgsender-read-it-directly)
  - [Cheaper to just read msg.sender rather than cache it](#cheaper-to-just-read-msgsender-rather-than-cache-it)
  - [Caching a global variable(msg.sender) is more expensive than reading it directly](#caching-a-global-variablemsgsender-is-more-expensive-than-reading-it-directly)
  - [We don't have to cache block.number](#we-dont-have-to-cache-blocknumber)
  - [Cheaper to read block.number directly here](#cheaper-to-read-blocknumber-directly-here)
- [Don't cache storage variable/External calls if using the cached value once.](#dont-cache-storage-variableexternal-calls-if-using-the-cached-value-once)
  - [VToken.sol.\_doTransferOut(): Don't cache `IERC20Upgradeable(underlying)`](#vtokensol_dotransferout-dont-cache-ierc20upgradeableunderlying)
  - [VToken.sol.\_getCashPrior(): token is being used once](#vtokensol_getcashprior-token-is-being-used-once)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Throughout the report some places might be denoted with audit tags to show the actual place affected.

## Massive 15k per tx gas savings - use 1 and 2 for Reentrancy guard
Using `true` and `false` will trigger gas-refunds, which after London are 1/5 of what they used to be, meaning using `1` and `2` (keeping the slot non-zero), will cost 5k per change (5k + 5k) vs 20k + 5k, saving you 15k gas per function which uses the modifier

[See solmate implementation](https://github.com/transmissions11/solmate/blob/main/src/utils/ReentrancyGuard.sol)

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L33-L38
```solidity
File: /contracts/VToken.sol
33:    modifier nonReentrant() {
34:        require(_notEntered, "re-entered");
35:        _notEntered = false;
36:        _;
37:        _notEntered = true; // get a gas-refund post-Istanbul
38:    }
```

**Note:** We could debate about the above issue being part of automated findings grouped under use of bools, however due to the huge impact it has I'm adding it here.

## The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L524-L531
### VToken.sol.sweepToken(): The results of owner() can be cached rather than call it twice
```solidity
File: /contracts/VToken.sol
524:    function sweepToken(IERC20Upgradeable token) external override {
525:        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");

528:        token.safeTransfer(owner(), balance);
```
In case of a revert on the first check we'd end up wasting about 3 gas used for the mstore but if we don't revert we'd save significant amount of gas
```diff
     function sweepToken(IERC20Upgradeable token) external override {
-        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
+        address _owner = owner();
+        require(msg.sender == _owner, "VToken::sweepToken: only admin can sweep tokens");
         require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
         uint256 balance = token.balanceOf(address(this));
-        token.safeTransfer(owner(), balance);
+        token.safeTransfer(_owner, balance);

         emit SweepToken(address(token));
     }
```


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L238-L242
### RiskFund.sol.\_swapAsset(): Results of ComptrollerViewInterface(comptroller).oracle() should be cached
```solidity
File: /contracts/RiskFund/RiskFund.sol
238:ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));//@audit: Initial call

242:        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
            address(vToken)
        );//@audit: 2nd call
```


## Looping on a storage variable
When using a for loop or any other loop, we should avoid iterating over storage variables as the loop can become very expensive. 
We should cache the storage variable in memory and use the cached value in the loop.


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L354-L361
### PoolRegistry.sol.getAllPools(): \_numberOfPools is a storage variable
```solidity
File: /contracts/Pool/PoolRegistry.sol
354:    function getAllPools() external view override returns (VenusPool[] memory) {
355:        VenusPool[] memory _pools = new VenusPool[](_numberOfPools);
356:        for (uint256 i = 1; i <= _numberOfPools; ++i) {
357:            address comptroller = _poolsByID[i];
358:            _pools[i - 1] = (_poolByComptroller[comptroller]);
359:        }
360:        return _pools;
361:    }
```

```diff
     function getAllPools() external view override returns (VenusPool[] memory) {
-        VenusPool[] memory _pools = new VenusPool[](_numberOfPools);
-        for (uint256 i = 1; i <= _numberOfPools; ++i) {
+        uint256 numberOfPools_ = _numberOfPools;
+        VenusPool[] memory _pools = new VenusPool[](numberOfPools_);
+        for (uint256 i = 1; i <= numberOfPools_; ++i) {
             address comptroller = _poolsByID[i];
             _pools[i - 1] = (_poolByComptroller[comptroller]);
         }
```


## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L257-L270
### RewardsDistributor.sol.updateContributorRewards(): Emit `contributorAccrued` instead of `rewardTokenAccrued[contributor]`
```solidity
File: /contracts/Rewards/RewardsDistributor.sol
257:    function updateContributorRewards(address contributor) public {

265:            rewardTokenAccrued[contributor] = contributorAccrued;
266:            lastContributorBlock[contributor] = blockNumber;

268:            emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
269:        }
270:    }
```

```diff
diff --git a/contracts/Rewards/RewardsDistributor.sol b/contracts/Rewards/RewardsDistributor.sol
index 434732d..d174c19 100644
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

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L151-L163
### Instead of reading from storage we can do the arithmetic inside the emit block
```solidity
File: /contracts/BaseJumpRateModelV2.sol
151:    function _updateJumpRateModel(
152:        uint256 baseRatePerYear,
153:        uint256 multiplierPerYear,
154:        uint256 jumpMultiplierPerYear,
155:        uint256 kink_
156:    ) internal {
157:        baseRatePerBlock = baseRatePerYear / blocksPerYear;
158:        multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
159:        jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
160:        kink = kink_;

162:        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
163:    }
```

```diff
diff --git a/contracts/BaseJumpRateModelV2.sol b/contracts/BaseJumpRateModelV2.sol
index 68b535a..0dc196e 100644
--- a/contracts/BaseJumpRateModelV2.sol
+++ b/contracts/BaseJumpRateModelV2.sol
@@ -159,7 +159,7 @@ abstract contract BaseJumpRateModelV2 is InterestRateModel {
         jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
         kink = kink_;

-        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
+        emit NewInterestParams(baseRatePerYear / blocksPerYear, (multiplierPerYear * BASE) / (blocksPerYear * kink_), jumpMultiplierPerYear / blocksPerYear, kink_);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/MaxLoopsLimitHelper.sol#L25-L32
### MaxLoopsLimitHelper.sol.\_setMaxLoopsLimit(): Emit `limit` instead of `maxLoopsLimit`
```solidity
File: /contracts/MaxLoopsLimitHelper.sol
25:    function _setMaxLoopsLimit(uint256 limit) internal {

29:        maxLoopsLimit = limit;

31:        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
```

```diff
diff --git a/contracts/MaxLoopsLimitHelper.sol b/contracts/MaxLoopsLimitHelper.sol
index 8607d70..f2ba93d 100644
--- a/contracts/MaxLoopsLimitHelper.sol
+++ b/contracts/MaxLoopsLimitHelper.sol
@@ -28,7 +28,7 @@ abstract contract MaxLoopsLimitHelper {
         uint256 oldMaxLoopsLimit = maxLoopsLimit;
         maxLoopsLimit = limit;

-        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
+        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, limit);
     }
```

## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L579

```solidity
File: /contracts/Comptroller.sol
579:        VToken[] memory userAssets = accountAssets[user];
```

```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..24841d2 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -576,7 +576,7 @@ contract Comptroller is
      * @custom:access Not restricted
      */
     function healAccount(address user) external {
-        VToken[] memory userAssets = accountAssets[user];
+        VToken[] storage userAssets = accountAssets[user];
         uint256 userAssetsCount = userAssets.length;
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L687
```solidity
File: /contracts/Comptroller.sol
687:        VToken[] memory borrowMarkets = accountAssets[borrower];

1304:        VToken[] memory assets = accountAssets[account];
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1441
```solidity
File: /contracts/VToken.sol
1441:        BorrowSnapshot memory borrowSnapshot = accountBorrows[account];
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L394
```solidity
File: /contracts/Pool/PoolRegistry.sol
394:        VenusPool memory venusPool = _poolByComptroller[comptroller];
```


## Use the cached value if it exists
If a cache value exists in a function, there is no need to read the storage value. Use the cached value as it would be cheaper.

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L190-L199
### Avoid reading from storage if a global variable would be equal to the variable being read
```solidity
File: /contracts/RiskFund/RiskFund.sol
190:    function transferReserveForAuction(address comptroller, uint256 amount) external override returns (uint256) {
191:        require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");

194:        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);
```
Note that in the above function, we require that `msg.sender == shortfall` which means the only way this function executes is if both sender and shortfail are the same. Thus whenever we need to reference `shortfall` which is a storage variable we can use the variable `msg.sender` as they both represent the same thing.
```diff
diff --git a/contracts/RiskFund/RiskFund.sol b/contracts/RiskFund/RiskFund.sol
index 32c3d30..3424827 100644
--- a/contracts/RiskFund/RiskFund.sol
+++ b/contracts/RiskFund/RiskFund.sol
@@ -191,7 +191,7 @@ contract RiskFund is
         require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");
         require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");
         poolReserves[comptroller] = poolReserves[comptroller] - amount;
-        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);
+        IERC20Upgradeable(convertibleBaseAsset).safeTransfer(msg.sender, amount);

         emit TransferredReserveForAuction(comptroller, amount);

```


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1368-L1369
### VToken.sol.\_initialize(): initialExchangeRateMantissa has been cached and the cached value should be used in the require statement
```solidity
File: /contracts/VToken.sol
1368:        initialExchangeRateMantissa = initialExchangeRateMantissa_;
1369:        require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..0def337 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1366,7 +1366,7 @@ contract VToken is

         // Set initial exchange rate
         initialExchangeRateMantissa = initialExchangeRateMantissa_;
-        require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
+        require(initialExchangeRateMantissa_ > 0, "initial exchange rate must be greater than zero.");
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1389-L1391
### Use the local variable `underlying_` instead of `underlying`
```solidity
File: /contracts/VToken.sol
1389:        // Set underlying and sanity check it
1390:        underlying = underlying_;
1391:        IERC20Upgradeable(underlying).totalSupply();
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..0ed239b 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1388,7 +1388,7 @@ contract VToken is

         // Set underlying and sanity check it
         underlying = underlying_;
-        IERC20Upgradeable(underlying).totalSupply();
+        IERC20Upgradeable(underlying_).totalSupply();

```

##  IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L726-L752
### Comptroller.sol.setCollateralFactor(): Cheaper to check function params before reading from storage
```solidity
File: /contracts/Comptroller.sol
726:    function setCollateralFactor(
727:        VToken vToken,
728:        uint256 newCollateralFactorMantissa,
729:        uint256 newLiquidationThresholdMantissa
730:    ) external {
731:        _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");

733:        // Verify market is listed
734:        Market storage market = markets[address(vToken)];
735:        if (!market.isListed) {
736:            revert MarketNotListed(address(vToken));
737:        }

739:        // Check collateral factor <= 0.9
740:        if (newCollateralFactorMantissa > collateralFactorMaxMantissa) {
741:            revert InvalidCollateralFactor();
742:        }

744:        // Ensure that liquidation threshold <= 1
745:        if (newLiquidationThresholdMantissa > mantissaOne) {
746:            revert InvalidLiquidationThreshold();
747:        }

749:        // Ensure that liquidation threshold >= CF
750:        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
751:            revert InvalidLiquidationThreshold();
752:        }
```
We have an if statement that checks the validity of some functional arguments. As this is cheaper compared to reading storage , we should reorder the if statements to have this check come first. In case of a revert on the functional argument checks, we wouldn't waste alot of gas

```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..b7a22ac 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -730,6 +730,11 @@ contract Comptroller is
     ) external {
         _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");

+        // Ensure that liquidation threshold >= CF
+        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
+            revert InvalidLiquidationThreshold();
+        }
+
         // Verify market is listed
         Market storage market = markets[address(vToken)];
         if (!market.isListed) {
@@ -746,11 +751,6 @@ contract Comptroller is
             revert InvalidLiquidationThreshold();
         }

-        // Ensure that liquidation threshold >= CF
-        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
-            revert InvalidLiquidationThreshold();
-        }
-
         // If collateral factor != 0, fail if price == 0
         if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
             revert PriceError(address(vToken));
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1018-L1057
### VToken.sol: Do all checks for function parameters first before making external call
```solidity
File: /contracts/VToken.sol
1026:        comptroller.preLiquidateHook(
1027:            address(this),
1028:            address(vTokenCollateral),
1029:            borrower,
1030:            repayAmount,
1031:            skipLiquidityCheck
1032:        );
        
1034:        if (accrualBlockNumber != _getBlockNumber()) {
1035:            revert LiquidateFreshnessCheck();
1036:        }

1040:        if (vTokenCollateral.accrualBlockNumber() != _getBlockNumber()) {
1041:            revert LiquidateCollateralFreshnessCheck();
1042:        }

1045:        if (borrower == liquidator) {
1046:            revert LiquidateLiquidatorIsBorrower();
1047:        }

1050:        if (repayAmount == 0) {
1051:            revert LiquidateCloseAmountIsZero();
1052:        }

1055:        if (repayAmount == type(uint256).max) {
1056:            revert LiquidateCloseAmountIsUintMax();
1057:        }
```
We have few checks for function parameters,consider having them come first as they are cheaper compared to making external calls.
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..81658ec 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1022,6 +1022,20 @@ contract VToken is
         VTokenInterface vTokenCollateral,
         bool skipLiquidityCheck
     ) internal {
+        /* Fail if borrower = liquidator */
+        if (borrower == liquidator) {
+            revert LiquidateLiquidatorIsBorrower();
+        }
+
+        /* Fail if repayAmount = 0 */
+        if (repayAmount == 0) {
+            revert LiquidateCloseAmountIsZero();
+        }
+
+        /* Fail if repayAmount = -1 */
+        if (repayAmount == type(uint256).max) {
+            revert LiquidateCloseAmountIsUintMax();
+        }
         /* Fail if liquidate not allowed */
         comptroller.preLiquidateHook(
             address(this),
@@ -1041,20 +1055,6 @@ contract VToken is
             revert LiquidateCollateralFreshnessCheck();
         }

-        /* Fail if borrower = liquidator */
-        if (borrower == liquidator) {
-            revert LiquidateLiquidatorIsBorrower();
-        }
-
-        /* Fail if repayAmount = 0 */
-        if (repayAmount == 0) {
-            revert LiquidateCloseAmountIsZero();
-        }
-
-        /* Fail if repayAmount = -1 */
-        if (repayAmount == type(uint256).max) {
-            revert LiquidateCloseAmountIsUintMax();
-        }

```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1097-L1109
### VToken.sol.\_seize(): Make function parameter checks first
```solidity
File: /contracts/VToken.sol
1097:    function _seize(

1104:        comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);

1107:        if (borrower == liquidator) {
1108:            revert LiquidateSeizeLiquidatorIsBorrower();
1109:        }
```
As borrower and liquidator are function arguments they are cheaper to check first. In case we revert on this check then we'd not waste a lot of gas making an external call
```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..67c58c4 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1100,13 +1100,14 @@ contract VToken is
         address borrower,
         uint256 seizeTokens
     ) internal {
-        /* Fail if seize not allowed */
-        comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);
-
         /* Fail if borrower = liquidator */
         if (borrower == liquidator) {
             revert LiquidateSeizeLiquidatorIsBorrower();
         }
+        /* Fail if seize not allowed */
+        comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1200-L1212
### VToken.sol.\_reduceReservesFresh(): Reorder the checks to have cheaper check first
```solidity
File: /contracts/VToken.sol
1200:    function _reduceReservesFresh(uint256 reduceAmount) internal {
1201:        // totalReserves - reduceAmount
1202:        uint256 totalReservesNew;

1205:        if (accrualBlockNumber != _getBlockNumber()) {
1206:            revert ReduceReservesFreshCheck();
1207:        }

1210:        if (_getCashPrior() < reduceAmount) {
1211:            revert ReduceReservesCashNotAvailable();
1212:        }
```
As it is, the first check reads from storage and then calls an internal function. The second check also has an internal function call but instead of reading from storage we read a function parameter which is cheaper compared to the storage read. The second check is cheaper than the first and we should reorder as follows.

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..e10d762 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1198,19 +1198,19 @@ contract VToken is
      * @param reduceAmount Amount of reduction to reserves
      */
     function _reduceReservesFresh(uint256 reduceAmount) internal {
-        // totalReserves - reduceAmount
-        uint256 totalReservesNew;
-
-        // We fail gracefully unless market's block number equals current block number
-        if (accrualBlockNumber != _getBlockNumber()) {
-            revert ReduceReservesFreshCheck();
-        }

         // Fail gracefully if protocol has insufficient underlying cash
         if (_getCashPrior() < reduceAmount) {
             revert ReduceReservesCashNotAvailable();
         }

+        // We fail gracefully unless market's block number equals current block number
+        if (accrualBlockNumber != _getBlockNumber()) {
+            revert ReduceReservesFreshCheck();
+        }
+        // totalReserves - reduceAmount
+        uint256 totalReservesNew;
+
         // Check reduceAmount ≤ reserves[n] (totalReserves)
         if (reduceAmount > totalReserves) {
             revert ReduceReservesCashValidation();
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1297-L1309
### VToken.sol.\_transferTokens(): Cheaper to check function params  than make external calls
```solidity
File: /contracts/VToken.sol
1297:    function _transferTokens(

1304:        comptroller.preTransferHook(address(this), src, dst, tokens);

1307:        if (src == dst) {
1308:           revert TransferNotAllowed();
1309:        }
```
The check for function argument should come before the external call. As we can revert if the function params don't meet some conditions , we can refactor the code as follows to have external call come after checking the function parameters
```diff
@@ -1300,13 +1300,13 @@ contract VToken is
         address dst,
         uint256 tokens
     ) internal {
-        /* Fail if transfer not allowed */
-        comptroller.preTransferHook(address(this), src, dst, tokens);
-
         /* Do not allow self-transfers */
         if (src == dst) {
             revert TransferNotAllowed();
         }
+
+        /* Fail if transfer not allowed */
+        comptroller.preTransferHook(address(this), src, dst, tokens);

```


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L157-L159
### Move checks for function arguments above the one reading storage
```solidity
File: /contracts/RiskFund/RiskFund.sol
157:        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
158:        require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
159:        require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");
```
The first require statement reads a variable from storage which is expensive compared to the other two require statements. Move the first require statement below the first two as those read function arguments
```diff
@@ -154,9 +154,10 @@ contract RiskFund is
         address[][] calldata paths
     ) external override returns (uint256) {
         _checkAccessAllowed("swapPoolsAssets(address[],uint256[],address[][])");
-        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
         require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
         require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");
+        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
+
```


## Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L59-L71
### Use calldata for `name_,symbol_ and riskManagement`
```solidity
File: /contracts/VToken.sol
59:    function initialize(
60:        address underlying_,
61:        ComptrollerInterface comptroller_,
62:        InterestRateModel interestRateModel_,
63:        uint256 initialExchangeRateMantissa_,
64:        string memory name_,
65:        string memory symbol_,
66:        uint8 decimals_,
67:        address admin_,
68:        address accessControlManager_,
69:        RiskManagementInit memory riskManagement,
70:        uint256 reserveFactorMantissa_
71:    ) external initializer {
```

**Other instances: check audit tags for what to use calldata on**
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L163-L167
```solidity
File: /contracts/Rewards/RewardsDistributor.sol
//@audit: Use calldata for marketBorrowIndex
163:    function distributeBorrowerRewardToken(
164:        address vToken,
165:        address borrower,
166:        Exp memory marketBorrowIndex
167:    ) external onlyComptroller {
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L197-L201
```solidity
File: /contracts/Rewards/RewardsDistributor.sol

//@audit: Use caldata for vTokens,supplySpeeds,borrowSpeeds
197:     function setRewardTokenSpeeds(
198:        VToken[] memory vTokens,
199:        uint256[] memory supplySpeeds,
200:        uint256[] memory borrowSpeeds
201:    ) external {
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L255
```solidity
File: /contracts/Pool/PoolRegistry.sol
//@audit: Use calldata on input
255:    function addMarket(AddMarketInput memory input) external {

//@audit: Use calldata on _metadata
343:    function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {
```
## Duplicated require()/revert() checks should be refactored to a modifier or function
This saves deployement gas

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L395-L397
```solidity
File: /contracts/VToken.sol
395:        if (msg.sender != address(comptroller)) {
396:            revert HealBorrowUnauthorized();
397:        }
```
The above check is also done on [Line 456](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L456-L458)


## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1223
```solidity
File: /contracts/VToken.sol
1223:        totalReservesNew = totalReserves - reduceAmount;
```
The operation `totalReserves - reduceAmount` cannot underflow due to the check on [Line 1215](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1215)that ensures that `totalReserves` is greater than `reduceAmount`
```diff
@@ -1220,7 +1220,9 @@ contract VToken is
         // EFFECTS & INTERACTIONS
         // (No safe failures beyond this point)

-        totalReservesNew = totalReserves - reduceAmount;
+        unchecked {
+           totalReservesNew =  totalReserves - reduceAmount;
+        }

```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L75
```solidity
File: /contracts/RiskFund/ProtocolShareReserve.sol
75:        poolsAssetsReserves[comptroller][asset] -= amount;
```
The operation `poolsAssetsReserves[comptroller][asset] - amount` cannot underflow due to the check on [Line 72](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L72) which ensures that `poolsAssetsReserves[comptroller][asset]` is greater than `amount`

## Nested if is cheaper than single statement using &&
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L754-L757
```solidity
File: /contracts/Comptroller.sol
754:        // If collateral factor != 0, fail if price == 0
755:        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
756:            revert PriceError(address(vToken));
757:        }
```

```diff
diff --git a/contracts/Comptroller.sol b/contracts/Comptroller.sol
index 41dc518..848a2ca 100644
--- a/contracts/Comptroller.sol
+++ b/contracts/Comptroller.sol
@@ -752,8 +752,10 @@ contract Comptroller is
         }

         // If collateral factor != 0, fail if price == 0
-        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
-            revert PriceError(address(vToken));
+        if (newCollateralFactorMantissa != 0){
+            if (oracle.getUnderlyingPrice(address(vToken)) == 0) {
+                revert PriceError(address(vToken));
+            }
         }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L837-L839
```solidity
File: /contracts/VToken.sol
837:        if (redeemTokens == 0 && redeemAmount > 0) {
838:            revert("redeemTokens zero");
839:        }
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..a44d059 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -834,8 +834,10 @@ contract VToken is
         }

         // Require tokens is zero or amount is also zero
-        if (redeemTokens == 0 && redeemAmount > 0) {
-            revert("redeemTokens zero");
+        if (redeemTokens == 0){
+            if (redeemAmount > 0) {
+                revert("redeemTokens zero");
+            }
         }

```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L462-L470
```solidity
File: /contracts/Lens/PoolLens.sol
462:        if (deltaBlocks > 0 && borrowSpeed > 0) {
...
470:        } else if (deltaBlocks > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L483-L490
```solidity
File: /contracts/Lens/PoolLens.sol
483:        if (deltaBlocks > 0 && supplySpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L506
```solidity
File: /contracts/Lens/PoolLens.sol
506:        if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

526:        if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L261
```solidity
File: /contracts/Rewards/RewardsDistributor.sol
261:        if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

348:        if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {

386:        if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {

418:        if (amount > 0 && amount <= rewardTokenRemaining) {

435:        if (deltaBlocks > 0 && supplySpeed > 0) {

463:        if (deltaBlocks > 0 && borrowSpeed > 0) {
```


## Don't cache global variables such as msg.sender

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L133-L140
### We don't need to cache msg.sender, read it directly
```solidity
File: /contracts/VToken.sol
133:    function approve(address spender, uint256 amount) external override returns (bool) {
134:        require(spender != address(0), "invalid spender address");

136:        address src = msg.sender;
137:        transferAllowances[src][spender] = amount;
138:        emit Approval(src, spender, amount);
139:        return true;
140:    }
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..7b602bc 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -133,9 +133,8 @@ contract VToken is
     function approve(address spender, uint256 amount) external override returns (bool) {
         require(spender != address(0), "invalid spender address");

-        address src = msg.sender;
-        transferAllowances[src][spender] = amount;
-        emit Approval(src, spender, amount);
+        transferAllowances[msg.sender][spender] = amount;
+        emit Approval(msg.sender, spender, amount);
         return true;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L625-L635
### Cheaper to just read msg.sender rather than cache it
```solidity
File: /contracts/VToken.sol
625:    function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {

628:        address src = msg.sender;
629:        uint256 newAllowance = transferAllowances[src][spender];
630:        newAllowance += addedValue;
631:        transferAllowances[src][spender] = newAllowance;

633:        emit Approval(src, spender, newAllowance);
634:        return true;
635:    }
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..c7894f1 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -625,12 +625,11 @@ contract VToken is
     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
         require(spender != address(0), "invalid spender address");

-        address src = msg.sender;
-        uint256 newAllowance = transferAllowances[src][spender];
+        uint256 newAllowance = transferAllowances[msg.sender][spender];
         newAllowance += addedValue;
-        transferAllowances[src][spender] = newAllowance;
+        transferAllowances[msg.sender][spender] = newAllowance;

-        emit Approval(src, spender, newAllowance);
+        emit Approval(msg.sender, spender, newAllowance);
         return true;
     }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L645-L659
### Caching a global variable(msg.sender) is more expensive than reading it directly
```solidity
File: /contracts/VToken.sol
648:        address src = msg.sender;
```


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L453-L472
### We don't have to cache block.number
```solidity
File: /contracts/Lens/PoolLens.sol
453:    function updateMarketBorrowIndex(

460:        uint256 blockNumber = block.number;
461:        uint256 deltaBlocks = sub_(blockNumber, uint256(borrowState.block));
462:        if (deltaBlocks > 0 && borrowSpeed > 0) {

469:            borrowState.block = safe32(blockNumber, "block number overflows");
470:        } else if (deltaBlocks > 0) {
471:            borrowState.block = safe32(blockNumber, "block number overflows");
472:        }
```


```diff
@@ -457,8 +457,7 @@ contract PoolLens is ExponentialNoError {
         Exp memory marketBorrowIndex
     ) internal view {
         uint256 borrowSpeed = rewardsDistributor.rewardTokenBorrowSpeeds(vToken);
-        uint256 blockNumber = block.number;
-        uint256 deltaBlocks = sub_(blockNumber, uint256(borrowState.block));
+        uint256 deltaBlocks = sub_(block.number, uint256(borrowState.block));
         if (deltaBlocks > 0 && borrowSpeed > 0) {
             // Remove the total earned interest rate since the opening of the market from total borrows
             uint256 borrowAmount = div_(VToken(vToken).totalBorrows(), marketBorrowIndex);
@@ -466,9 +465,9 @@ contract PoolLens is ExponentialNoError {
             Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });
             Double memory index = add_(Double({ mantissa: borrowState.index }), ratio);
             borrowState.index = safe224(index.mantissa, "new index overflows");
-            borrowState.block = safe32(blockNumber, "block number overflows");
+            borrowState.block = safe32(block.number, "block number overflows");
         } else if (deltaBlocks > 0) {
-            borrowState.block = safe32(blockNumber, "block number overflows");
+            borrowState.block = safe32(block.number, "block number overflows");
         }
     }

```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L481
### Cheaper to read block.number directly here
```solidity
File: /contracts/Lens/PoolLens.sol
481:        uint256 blockNumber = block.number;
```

## Don't cache storage variable/External calls if using the cached value once.

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1284-L1287
### VToken.sol.\_doTransferOut(): Don't cache `IERC20Upgradeable(underlying)`

```solidity
File: /contracts/VToken.sol
1284:    function _doTransferOut(address to, uint256 amount) internal virtual {
1285:        IERC20Upgradeable token = IERC20Upgradeable(underlying);
1286:        token.safeTransfer(to, amount);
1287:    }
```

```diff
     function _doTransferOut(address to, uint256 amount) internal virtual {
-        IERC20Upgradeable token = IERC20Upgradeable(underlying);
-        token.safeTransfer(to, amount);
+        IERC20Upgradeable(underlying).safeTransfer(to, amount);
     }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1421-L1424
### VToken.sol.\_getCashPrior(): token is being used once
```solidity
File: /contracts/VToken.sol
1421:    function _getCashPrior() internal view virtual returns (uint256) {
1422:        IERC20Upgradeable token = IERC20Upgradeable(underlying);
1423:        return token.balanceOf(address(this));
1424:    }
```

```diff
diff --git a/contracts/VToken.sol b/contracts/VToken.sol
index 8d9a2dc..5431de1 100644
--- a/contracts/VToken.sol
+++ b/contracts/VToken.sol
@@ -1419,8 +1419,7 @@ contract VToken is
      * @return The quantity of underlying tokens owned by this contract
      */
     function _getCashPrior() internal view virtual returns (uint256) {
-        IERC20Upgradeable token = IERC20Upgradeable(underlying);
-        return token.balanceOf(address(this));
+        return IERC20Upgradeable(underlying).balanceOf(address(this));
     }
```


