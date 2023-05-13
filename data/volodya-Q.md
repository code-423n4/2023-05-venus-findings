| Number | Optimization Details                                                                                      | Context |
| :----: | :-------------------------------------------------------------------------------------------------------- | :-----: |
| [L-01] | Users can borrow up to the borrowCap amount, but they should borrow less than that. |   2    |
| [N-01] | Users might not have the ability to exit before a critical change takes place |    1    |

Total 3 issues
### [L-01] Users can borrow up to the borrowCap amount, but they should borrow less than that.

According to the docs from the code function `Borrowing that brings total borrows to borrow cap will revert`
```solidity
    /**
     * @notice Set the given borrow caps for the given vToken markets. Borrowing that brings total borrows to or above borrow cap will revert.
...
```
[contracts/Comptroller.sol#L839](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L839)

 this means that whevener user should not be able to hit borrowCap and revert if `nextTotalBorrows == borrowCap` but it doesn't
```solidity
        if (borrowCap != type(uint256).max) {
            uint256 totalBorrows = VToken(vToken).totalBorrows();
            uint256 nextTotalBorrows = totalBorrows + borrowAmount;
            //          @audit should be, users can borrow more than allowed
            //            if (nextTotalBorrows >= borrowCap) {
            if (nextTotalBorrows > borrowCap) {
                revert BorrowCapExceeded(vToken, borrowCap);
            }
        }
```
[contracts/Comptroller.sol#L354](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L354)
## Recommended Mitigation Steps
Same issue with supplyCap or change documentation
```diff
        if (borrowCap != type(uint256).max) {
            uint256 totalBorrows = VToken(vToken).totalBorrows();
            uint256 nextTotalBorrows = totalBorrows + borrowAmount;
-            if (nextTotalBorrows > borrowCap) {
+            if (nextTotalBorrows >= borrowCap) {
                revert BorrowCapExceeded(vToken, borrowCap);
            }
        }
```

```diff
        if (supplyCap != type(uint256).max) {
            uint256 vTokenSupply = VToken(vToken).totalSupply();
            Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
            uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);
-            if (nextTotalSupply > supplyCap) {
+            if (nextTotalSupply >= supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
        }
```


### [N-01] Users might not have the ability to exit before a critical change takes place

According to report by [openzeppelin](https://blog.openzeppelin.com/compound-finance-patch-audit/)
>  two functions users may need to exit Compound (namely `redeem` and `repayBorrow`) always available to users, so users have the ability to exit before a critical change takes place. 

In current implementation protocol introduced ways to forbid users to exit protocol.
```solidity
    function preRepayHook(address vToken, address borrower) external override {
        _checkActionPauseState(vToken, Action.REPAY);
...
}
```
[contracts/Comptroller.sol#L391](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L391)

```solidity
    function preRepayHook(address vToken, address borrower) external override {
        _checkActionPauseState(vToken, Action.REPAY);
...
}
```
[contracts/Comptroller.sol#L391](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L391)

```solidity
    function preRedeemHook(
        address vToken,
        address redeemer,
        uint256 redeemTokens
    ) external override {
        _checkActionPauseState(vToken, Action.REDEEM);
....
```
[contracts/Comptroller.sol#L297](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L297)

## Recommended Mitigation Steps
Remove those checks if you would like to follow compound principals for user`s convenience.

```diff
    function preRepayHook(address vToken, address borrower) external override {
-        _checkActionPauseState(vToken, Action.REPAY);

        oracle.updatePrice(vToken);

        if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }

        // Keep the flywheel moving
        uint256 rewardDistributorsCount = rewardsDistributors.length;

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
        }
    }

```

```diff
    function preRedeemHook(
        address vToken,
        address redeemer,
        uint256 redeemTokens
    ) external override {
-        _checkActionPauseState(vToken, Action.REDEEM);
        oracle.updatePrice(vToken);
        _checkRedeemAllowed(vToken, redeemer, redeemTokens);

        // Keep the flywheel moving
        uint256 rewardDistributorsCount = rewardsDistributors.length;

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, redeemer);
        }
    }

```
