## Low Issues
### L-1 VToken.sol_transfer to `address(0)` is possible
[`transfer`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L99) and [`transferFrom`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L114) functions have no checks for `address(0)` recipient but [`mintBehalf`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L195-L196C65) function has one. While minting on `address(0)` is restricted transfering is possible. I suggest adding a corresponding check.

### L-2 VToken.sol_transfer zero amount is possible
[`transfer`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L99) and [`transferFrom`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L114) functions have no checks for zero amount. I suggest adding corresponding checks if it is necessary.

### L-3 Shortfall.sol_There is no check for `waitForFirstBidder` new value
[`updateWaitForFirstBidder`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L334) has no check for zero value but [`updateNextBidderBlockLimit`](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL293C14-L293C40) has one. I suggest adding a corresponding check.

## Non Critical
### N-1 Typo in comments
There should be `an` instead of `a`.
```solidity
     * @notice Start a auction when there is not currently one active
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L356

### N-2 `constant` should be defined rather than using magic number
```solidity
1055:        if (repayAmount == type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1055C1-L1055C48