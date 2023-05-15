## Redundant Assignment of rewardsDistributors.length 
An initial assignment of rewardsDistributors.length into the memory variable rewardDistributorsLength is made on line 930.

[Comptroller.sol#L930 - rewardDistributorsLength](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L930)

Another variable rewardsDistributorsLen is then subsequently assigned to the same value rewardsDistributors.length, adding unnecessary memory allocation and storage operations. 

[Comptroller.sol#L940 - rewardsDistributorsLen](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L940)

It is recommended to remove the unnecessary operation to save the gas associated with costly **SLOAD (800)** operations.