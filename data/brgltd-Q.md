| ID    | Title |
| -------- | ------- |
| 01 | `PoolRegistry.supportMarket()` cannot be paused
| 02 | Lack of revert if price returned from oracle is zero
| 03 | Solidity version
| 04 | State update after external calls
| 05 | Check for stale values on setter functions
| 06 | Variable shadow
| 07 | `__gap` could be declared on a single file to be reused across the project
| 08 | Consistent usage of require vs custom error
| 09 | Avoid duplicated computation in `Comptroller.addRewardsDistributor()`
| 10 | Eslint warning in a solidity file
| 11 | Interchangeable usage of msg.sender and vToken in in `Comptroller.preBorrowCheck()`
| 12 | Using underscore in a single struct field
| 13 | Uncommented fields in a struct
| 14 | Use return named variables or explicit returns consistently

## [01] `PoolRegistry.supportMarket()` cannot be paused

Most actions in `PoolRegistry` can be paused, e.g. `_addToMarket()`, `exitMarket()`, `preMintHook()` etc.

Consider adding a `support` field in the enum Action and allowing `supportMarket()` to be paused. Not allowing `supportMarket()` to be paused could result in irregular behavior if everything else (mint, redeeming, borrowing enter market, exit market etc) is paused but `supportMarket()` is open.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L801-L824

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1177

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L188

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L254

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L44-L54

## [02] Lack of revert if price returned from oracle is zero

`RiskFund._swapAsset()` will not revert if `getUnderlyingPrice` if the price return zero.

This might be intended to avoid making the loop in `RiskFund.swapPoolAssets()` revert if a single swap is not done. However, it might be beneficial to revert the entire transaction, since price zero means the price is unavailable.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L174

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L240-L242

https://github.com/VenusProtocol/venus-protocol/blob/e085f1194bd942c2e75de5787a0a84ec274c6dd4/contracts/Oracle/PriceOracle.sol#L13

## [03] Solidity version

All contracts are using 0.8.13. Consider updating to the latest version 0.8.19 to ensure the compiler contains the latest security fixes.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L2

## [04] State update after external calls

Consider make state updates prior of executing external calls to follow the checks-effects-interactions pattern.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L183

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L187

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L190

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L193

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L197-L199

## [05] Check for stale values on setter functions

Add a check ensuring that the new value if different than the old value to avoid emitting unnecessary events.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L702-L710

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L779-L792

## [06] Variable shadow

Consider renaming the input `accessControlManager` in `PoolRegistry.initialize()`. Currently it's being shadowed by `AccessControlledV8.accessControlManager()`. This will get a warning on common linters/text editors.

Also consider renaming the input `owner` in `VToken.allowance()` and `VToken.balanceOf()`, since it's being shadowed by `OwnableUpgradeable.owner()`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L170

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L539

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L548

## [07] `__gap` could be declared in a single file to be reused across the project

To improve code reusability, consider declaring gap on a single file to be reused.Please, refer to this [example](https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/mixins/shared/Gap10000.sol) for an implementation reference.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L122

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L130

## [08] Consistent usage of require vs custom error

Consider using the same approach throughout the codebase to improve the consistency of the code.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L396

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L489-L490

## [09] Avoid duplicated computation in `Comptroller.addRewardsDistributor()`

`uint256 rewardsDistributorsLen = rewardsDistributors.length;` can be removed and the `rewardsDistributorsLength` from L930 can be reused.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L940

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L930

## [10] Eslint warning in a solidity file

The comment on `Comptroller.preBorrowHook()` seems to have been intended for a js/ts test file. 

Consider removing from the solidity source code or add a comment on why it needs to be there.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L323

## [11] Interchangeable usage of msg.sender and vToken in in `Comptroller.preBorrowCheck()`

Consider replacing `_addToMarket(VToken(msg.sender), borrower)` with `_addToMarket(VToken(vToken), borrower)`, since `msg.sender` will have to be equal `vToken` for `_checkSender(vToken)` to pass. This can improve code clarity.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L339-L342

## [12] Using underscore in a single struct field

Consider refactoring `kink_` to `kink`.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L43

## [13] Uncommented fields in a struct

Consider adding comments for all the fields in a struct to improve the readability of the codebase.

Example with comments:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L29-L42

Example without comments

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L8-L27

# [14] Use return named variables or explicit returns consistently

Some functions are declaring returned named variables on the function header, while other functions are not defining return named variables.

Following function does not declare a returned named variable.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L379

Following function is using a return named variable in the function header.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L222

Consider adopting the same approach throughout the codebase to improve the explicitness and readability of the code.
