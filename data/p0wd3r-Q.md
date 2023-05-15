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

# The `input.initialSupply` in addMarket needs to be validated as not equal to 0.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L321
```solidity
        IERC20Upgradeable token = IERC20Upgradeable(input.asset);
        uint256 amountToSupply = _transferIn(token, msg.sender, input.initialSupply);
        token.safeApprove(address(vToken), 0);
        token.safeApprove(address(vToken), amountToSupply);
        vToken.mintBehalf(input.vTokenReceiver, amountToSupply);
```
When `addMarket` is executed, a certain amount of vToken will be minted first, with the quantity being `input.initialSupply`. 
However, input does not restrict it from being 0. If it is 0, then the initial minting process will not be completed and this will lead to the classic "First deposit bug" in cToken: https://github.com/code-423n4/2023-01-ondo-findings/issues/247