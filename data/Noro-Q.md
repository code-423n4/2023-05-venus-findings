# Contradiction between the docs comments and the code (docs incorrect)

[In ComptrollerStorage in the Market struct](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L38) : it says that liquidationThreshold Must be between 0 and collateral factor .

But in [Comptroller#setCollateralFactor()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L749C1-L752) if we set the collateral factor < liquidation threshold it will revert .

Thus, `collateral factor` should always be **less than**  `liquidation threshold`, unlike what the comment says .

