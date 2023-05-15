1. uncheck ++i;

instead of 
```solidity

        for (uint256 i; i < marketsCount; ++i) {
            ...
        }
```
do
```solidity
        for (uint256 i; i < marketsCount;) {
            ...
           unchecked {
               ++i
        }
```

There are 28 places across ShortFall, RewardDistributor, RiskFund, PoolRegistry and Comptroller that can be improved.