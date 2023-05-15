# Code4rena Venus Protocol May 2023 - Team_Rocket Gas Optimizations Report

### [G-01] Calling `safeApprove` twice is unneccessary

#### Location

[PoolRegistry.sol#L322-L323](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L322-L323)

[RiskFund.sol#L258-L259](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L258-L259)

#### Description

`safeApprove` is called twice each time to safely approve the `VToken` contract to spend the underlying asset, and the PancakeSwap Router to swap assets. Using `safeApprove` is unnecessary as the allowance is spent in the same transaction that the approval occurs, meaning that there will never be leftover allowance after a transaction that can be exploited.

If a scenario where there is actually leftover allowance were to occur, `safeApprove` will not prevent a front-running attack from occurring since the call with `amount = 0` is called in the same transaction as the call with `amount > 0`.

#### Recommended Steps

Just call `approve` once with the required `amount`.

```diff
@@ -319,8 +319,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist

         IERC20Upgradeable token = IERC20Upgradeable(input.asset);
         uint256 amountToSupply = _transferIn(token, msg.sender, input.initialSupply);
-        token.safeApprove(address(vToken), 0);
-        token.safeApprove(address(vToken), amountToSupply);
+        token.approve(address(vToken), amountToSupply);
         vToken.mintBehalf(input.vTokenReceiver, amountToSupply);

         emit MarketAdded(address(comptroller), address(vToken));

@@ -255,8 +255,7 @@ contract RiskFund is
                         path[path.length - 1] == convertibleBaseAsset,
                         "RiskFund: finally path must be convertible base asset"
                     );
-                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
-                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);
+                    IERC20Upgradeable(underlyingAsset).approve(pancakeSwapRouter, balanceOfUnderlyingAsset);
                     uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
                         balanceOfUnderlyingAsset,
                         amountOutMin,
```

### [G-02] Use `calldata` arrays instead of `memory` arrays as function parameters wherever possible

#### Location

[Comptroller.sol#L154](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154)

#### Description

Using a `memory` array incurs extra gas costs as there is extra overhead relating to copying the calldata onto memory before adding it onto the EVM stack to access its elements. This is not the case with `calldata` arrays, as the array is loaded from calldata directly onto the stack using the `CALLDATALOAD` opcode. In situations where the array does not need to be modified, it is more gas efficient to use a `calldata` array.

#### Recommended Steps

Change the `vTokens` type from `address[] memory` to `address[] calldata`.

```diff
@@ -151,7 +151,7 @@ contract Comptroller is
      * @custom:error MarketNotListed error is thrown if any of the markets is not listed
      * @custom:access Not restricted
      */
-    function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {
+    function enterMarkets(address[] calldata vTokens) external override returns (uint256[] memory) {
         uint256 len = vTokens.length;

         uint256 accountAssetsLen = accountAssets[msg.sender].length;
```

### [G-03] Cheaper checks should go before more expensive checks

#### Location

[Comptroller.sol#L190-L206](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L190-L206)

#### Description

The `exitMarket` function performs a series of checks to make sure that the user is allowed to exit the market. It should check if the sender is not already in the market first, to avoid unnecessary expensive calls to `_safeGetAccountSnapshot` and `_checkRedeemAllowed` in the case that it is true.

#### Recommended Steps

Check if the sender is not already in the market before getting a snapshot of their account.

```diff
@@ -187,6 +187,12 @@ contract Comptroller is
     function exitMarket(address vTokenAddress) external override returns (uint256) {
         _checkActionPauseState(vTokenAddress, Action.EXIT_MARKET);
         VToken vToken = VToken(vTokenAddress);
+
+        /* Return true if the sender is not already ‘in’ the market */
+        if (!marketToExit.accountMembership[msg.sender]) {
+            return NO_ERROR;
+        }
+
         /* Get sender tokensHeld and amountOwed underlying from the vToken */
         (uint256 tokensHeld, uint256 amountOwed, ) = _safeGetAccountSnapshot(vToken, msg.sender);

@@ -200,11 +206,6 @@ contract Comptroller is

         Market storage marketToExit = markets[address(vToken)];

-        /* Return true if the sender is not already ‘in’ the market */
-        if (!marketToExit.accountMembership[msg.sender]) {
-            return NO_ERROR;
-        }
-
         /* Set vToken account membership to false */
         delete marketToExit.accountMembership[msg.sender];
```

### [G-04] Assign values to storage directly instead of memory first

#### Location

[ VToken.sol#L1319-L1328 ](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1319-L1328)

#### Description

Math is performed and assigned to a memory variable before it's assigned to a storage variable to check for under/overflows. This is unnecessary, as these checks are also done if the storage variable was directly performed on. You can avoid the gas costs of storing data into memory by just performing the math directly on the storage variable.

#### Recommended Steps

Perform math directly on storage variables.

```diff
@@ -1316,20 +1316,15 @@ contract VToken is
             startingAllowance = transferAllowances[src][spender];
         }

-        /* Do the calculations, checking for {under,over}flow */
-        uint256 allowanceNew = startingAllowance - tokens;
-        uint256 srcTokensNew = accountTokens[src] - tokens;
-        uint256 dstTokensNew = accountTokens[dst] + tokens;
-
         /////////////////////////
         // EFFECTS & INTERACTIONS

-        accountTokens[src] = srcTokensNew;
-        accountTokens[dst] = dstTokensNew;
+        accountTokens[src] = accountTokens[src] - tokens;
+        accountTokens[dst] = accountTokens[dst] + tokens;

         /* Eat some of the allowance (if necessary) */
         if (startingAllowance != type(uint256).max) {
-            transferAllowances[src][spender] = allowanceNew;
+            transferAllowances[src][spender] = startingAllowance - tokens;
         }

         /* We emit a Transfer event */
```
