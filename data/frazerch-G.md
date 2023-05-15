# [G-01] Redundant cache of storage variable into a local variable

Inside the function `addRewardsDistributor` the length of `rewardsDistributors` is retrieved from storage and initially stored inside a local variable `rewardsDistributorsLength`.

Inside the same function, the length of `rewardsDistributors` is then again retrieved from storage and stored inside a new local variable `rewardsDistributorsLen`. The length of the `rewardsDistributors` is not affected, hence the second assignment of the length is not needed and consumes unnecessary gas

```
File: contracts/Comptroller.sol

930:    	uint256 rewardsDistributorsLength = rewardsDistributors.length;
940:    	uint256 rewardsDistributorsLen = rewardsDistributors.length;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L930 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L940 
