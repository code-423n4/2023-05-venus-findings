## [QA-01] Inconsistency in docs and code about the shortfall contract

The docs mentions that "If the pool’s bad debt exceeds the risk fund plus a 10% incentive", but in the code it looks like it checks if the pools bad debt plus 10% incentive exceeds the risk fund. 

```solidity
uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);
```

The docs need to be updated, as I asked the author and they said that the code is the correct implementation.