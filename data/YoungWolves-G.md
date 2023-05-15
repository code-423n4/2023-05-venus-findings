# [G-01] Use named parameters in returns

#### File: 
**WhitePaperInterestRateModelFactory**, **JumpRateModelFactory**

#### Description:
In the deploy() function of both contracts it is returned its respective model. If we use a named parameter instead of using the return keyword, some extra gas can be saved.

Gas saved per instance: 13 gas


# [G-02] Use !=0 instead of > 0 for uint256 varialbles

#### File:
**Comptroller.sol:** L367, 1262
**VToken.sol:** L818, 837,1369
**PoolLens.sol:** L462, 466, 470, 483, 486, 490, 506, 526
**RewardDistributor.sol:** L261, 418, 435, 438, 446, 463, 466, 474
**RiskFund.sol:** L82, 83, 139, 244
**MockPancackeSwap.sol:** L469, 470, 480, 481, 494, 495, 525

#### Description:
Operation GT(0x11) costs 3 gas more than using a non-equal operator. And considering the big amount of places this is used, the amount of gas saved will be important.


# [G-03] Remove duplicate rewardsDistributors.lengthcaching

#### File:
**Comptroller**: L930, 940

```
function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner {
    require(!rewardsDistributorExists[address(_rewardsDistributor)], "already exists");

    -> uint256 rewardsDistributorsLength = rewardsDistributors.length;

    for (uint256 i; i < rewardsDistributorsLength; ++i) {
        address rewardToken = address(rewardsDistributors[i].rewardToken());
        require(
            rewardToken != address(_rewardsDistributor.rewardToken()),
            "distributor already exists with this reward"
        );
    }

    -> uint256 rewardsDistributorsLen = rewardsDistributors.length;
    _ensureMaxLoops(rewardsDistributorsLen + 1);
```