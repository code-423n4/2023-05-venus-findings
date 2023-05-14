## [Low - 1] Oracle zero price check is not always done

Examples of when it is checked:
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L345-L347
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755C10-L757
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1369C1-L1372

And when it is not:
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L240C8-L242
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L393
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L266-L268
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L405-L409

Submitted as "Low" because the oracle is OOS.

## [Low - 2] `NO_ERROR` check is not always handled

The `NO_ERROR` constant is returned to notify that the execution of the last call, even if didn't reverted, went smoothly. This should be handled.
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL158C11-L158C11
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L168
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L181
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L198
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L214
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L227
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L242
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L256
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L271
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L333
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L346
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L357
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L372
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL666C1-L666C1
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L996

## [Low - 3] Wrong emitted event

In the `healBorrow()` function of the `VToken` contract, the `HealBorrow()` event is emitted at the end: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L428
But the last element appears to be wrong. It emits `repayAmount`, but we probably need `actualRepayAmount`, in the case of using fee-on-transfer tokens.

## [Low - 4] `delete` on a dynamic type does not zeroes out all the elements

In the `Shortfall` contract, when creating a new auction, we must empty the old values of the last one.
This is done by cleaning the `Auction` struct with multiple tricks.
When it comes to cleaning the array `market` field, the `delete` keyword is used: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#LL379C13-L379C13
But using the `delete` keyword on a dynamic type only resets its length to 0.
Luckily, this doesn't seem to have any bad consequences but it could have got under different circumstances.

## [NC - 1] Needless address check

In the `_liquidateBorrowFresh()` function of the `VToken` contract used to liquidate underwater accounts, there is a check to make sure that the liquidator isn't the borrower at the same time: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL1046C48-L1046C48
It seems like this check is useless because it is already done in the `seize()` function that is called later on.