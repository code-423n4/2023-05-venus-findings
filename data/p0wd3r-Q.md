# `_addReservesFresh` return value is not being used

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1177
```solidity
function _addReservesFresh(uint256 addAmount) internal returns (uint256) {
```

The only place where this function is used is as follows, and its return value is not used.
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L356-L359
```solidity
    function addReserves(uint256 addAmount) external override nonReentrant {
        accrueInterest();
        _addReservesFresh(addAmount);
    }
```