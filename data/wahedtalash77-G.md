## [G-1] For uint use != 0 instead of > 0
367: `  if (snapshot.shortfall > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L367

1262: `if (snapshot.shortfall > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1262

818: `if (redeemTokensIn > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L818

837: `if (redeemTokens == 0 && redeemAmount > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837

261: ` if (deltaBlocks > 0 && rewardTokenSpeed > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261

435: ` if (deltaBlocks > 0 && supplySpeed > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L435

446: ` } else if (deltaBlocks > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L446

463: ` if (deltaBlocks > 0 && borrowSpeed > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L463

474: ` } else if (deltaBlocks > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L474



## [G-2] you can save storage by reordering Holding struct fields in the following way.
35:
   ```
struct VTokenMetadata {
        uint256 exchangeRateCurrent;
        uint256 supplyRatePerBlock;
        uint256 borrowRatePerBlock;
        uint256 reserveFactorMantissa;
        uint256 supplyCaps;
        uint256 borrowCaps;
        uint256 totalBorrows;
        uint256 totalReserves;
        uint256 totalSupply;
        uint256 totalCash;
        address vToken;
        uint256 collateralFactorMantissa;
        uint256 vTokenDecimals;
        uint256 underlyingDecimals;
        address underlyingAssetAddress;
        bool isListed;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L35

57:
```
struct VTokenBalances {
        uint256 balanceOf;
        uint256 borrowBalanceCurrent;
        uint256 balanceOfUnderlying;
        uint256 tokenBalance;
        uint256 tokenAllowance;
        address vToken;
    }
```
	
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L57

69:
   ```
struct VTokenUnderlyingPrice {
        uint256 underlyingPrice;
        address vToken;
    }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L69

77:
 ```
struct PendingReward {
        uint256 amount;
        address vTokenAddress;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L77

85:
   ```
struct RewardSummary {
        uint256 totalRewards;
        address distributorAddress;
        address rewardTokenAddress;
        PendingReward[] pendingRewards;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L85

105:
   ```
struct BadDebt {
        uint256 badDebtUsd;
        address vTokenAddress;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L105

113: 
  ```
struct BadDebtSummary {
        uint256 totalBadDebtUsd;
        address comptroller;
        BadDebt[] badDebts;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L113

and in other place where they use struct.

## [G-03] Avoid unnecessary read of array length in for loops can save gas
418:  `for (uint256 i; i < markets.length; ++i) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L418

229: `for (uint256 i; i < rewardsDistributors.length; ++i) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L229

## [G-4] Use double if statements instead of &&
### Using nested if cheaper than using && multiple check combinations.

1166:  `markets[address(vToken)].collateralFactorMantissa == 0 &&` 
1167:  `actionPaused(address(vToken), Action.BORROW) &&`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL1166C11-L1167C60

462: `if (deltaBlocks > 0 && borrowSpeed > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462

483 :   `if (deltaBlocks > 0 && supplySpeed > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#LL483C45-L483C45

506: `if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#LL506C16-L506C16

526: `if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#LL526C9-L526C71


## [G-5]  we shold choose less range instead of `uint256` like `uint8` so the best way is to put them in struct. It should be noted that in a struct, uint8 DOES cost less than a traditional uint, because of the tight packing feature. Also be sure that your uints are next to your other uints, 

103: `uint256 internal constant NO_ERROR = 0;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103

109: `uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L109

112: `uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L112

18: ` uint256 private constant protocolSharePercentage = 70;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L18

19: ` uint256 private constant baseUnit = 100;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L19

17: ` uint256 public constant blocksPerYear = 2102400;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L17

5: `  uint256 public constant NO_ERROR = 0;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L5

## [G-6] Use solidity version 0.8.19 to gain some gas boost
>Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
> 
>https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManager.sol#L2
>
>https://github.com/code-423n4/2023-04-eigenlayer/blob/main/src/contracts/core/StrategyManagerStorage.sol#L2
> 
>...
>See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

