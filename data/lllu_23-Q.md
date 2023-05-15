# [N-01] `Variables` need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.

`File: contracts/ComptrollerStorage.sol`
`103: uint256 internal constant NO_ERROR = 0;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103

# [N-02] Use nested `if` and avoid multiple check combinations

There are 6 instances of this issue:

`File: contracts/Rewards/RewardsDistributor.sol`
`348: if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex)`
`386:      if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex)`
`418:        if (amount > 0 && amount <= rewardTokenRemaining`
`435:   if (deltaBlocks > 0 && supplySpeed > 0)`
`463:       if (deltaBlocks > 0 && borrowSpeed > 0)`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L348

`File: contracts/Lens/PoolLens.sol`
`526: if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0)`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526
