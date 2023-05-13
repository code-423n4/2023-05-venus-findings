| Number | Optimization Details                                                                                      | Context |
| :----: | :-------------------------------------------------------------------------------------------------------- | :-----: |
| [N-01] | Users might not have the ability to exit before a critical change takes place |    2    |
| [G-02] | Setting the constructor to payable payable                                                                |   21    |
| [G-03] | Redundant variables                                                                                       |   4    |
| [G-04] | Reorder the `if` statements for `revert` to have the less gas consuming before the expensive one          |   4   |
| [G-05] | use nested if instead of `&&`                                                 |   12    |
| [G-06] | Empty blocks should be removed or emit something         |   2   |
| [G-07] | Using unchecked blocks to save gas |   1   |
| [G-08] | Caching global variables is more expensive than using the actual variable(use msg.sender instead of caching it |   2   |
| [G-09] | Optimize names to save gas                                                           |    All files    |

Total 48 issues

### [N-01] Users might not have the ability to exit before a critical change takes place

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.
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

## Tools Used

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
