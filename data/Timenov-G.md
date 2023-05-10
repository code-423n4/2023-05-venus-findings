# Venus report by Timenov

## Summary
G-01 Using `x += y` costs more gas than using `x = x + y`.
G-02 Using `> 0` costs more than using `!= 0`.

### [G-01] Using `x += y` costs more gas than using `x = x + y`.
*There are 4 instances of this issue.*

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

74: assetsReserves[asset] -= amount;
75: poolsAssetsReserves[comptroller][asset] -= amount;
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L74-L75

```solidity
File: contracts/RiskFund/RiskFund.sol

249: assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
250: poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L249-L250

### [G-02] Using `> 0` costs more than using `!= 0`.
*There are 26 instances of this issue.*

```solidity
File: contracts/Comptroller.sol

367: snapshot.shortfall > 0
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L367

```solidity
File: contracts/VToken.sol

818: redeemTokensIn > 0
837: redeemAmount > 0
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L818
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L837

```solidity
File: contracts/Lens/PoolLens.sol

462: deltaBlocks > 0 && borrowSpeed > 0
470: deltaBlocks > 0
483: deltaBlocks > 0 && supplySpeed > 0
490: deltaBlocks > 0
506: borrowIndex.mantissa > 0
526: supplyIndex.mantissa > 0
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL462C13-L462C47
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL470C20-L470C35
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL483C13-L483C47
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL490C20-L490C35
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL506C44-L506C68
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL526C44-L526C68

```solidity
File: contracts/Rewards/RewardsDistributor.sol

261: deltaBlocks > 0 && rewardTokenSpeed > 0
418: amount > 0
435: deltaBlocks > 0 && supplySpeed > 0
438: supplyTokens > 0
446: deltaBlocks > 0
463: deltaBlocks > 0 && borrowSpeed > 0
466: borrowAmount > 0
474: deltaBlocks > 0
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL261C13-L261C52
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL418C13-L418C23
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL435C13-L435C47
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL438C35-L438C51
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL446C20-L446C35
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL463C13-L463C47
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL466C35-L466C51
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL474C20-L474C35

```solidity
File: contracts/RiskFund/RiskFund.sol

82: minAmountToConvert_ > 0
83: loopsLimit_ > 0
139: minAmountToConvert_ > 0
244: balanceOfUnderlyingAsset > 0
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L82-L83
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL139C17-L139C40
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL244C13-L244C41