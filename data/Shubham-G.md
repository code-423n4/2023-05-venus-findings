## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | Sort Solidity operations using short-circuit mode | 1 |
| [GAS-02](#GAS-02) | Inverting the condition of an if-else-statement saves gas | 4 |


## [GAS-01] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
File: main/contracts/Shortfall/Shortfall.sol

L:365        (auction.startBlock == 0 && auction.status == AuctionStatus.NOT_STARTED) ||
                auction.status == AuctionStatus.ENDED,
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#LL365C13-L366C55


## [GAS-02] Inverting the condition of an if-else-statement saves gas

Flipping the true and false blocks instead saves gas. 
If the comparison is true then first condition is evaluated which calls another function & requires more gas to compute the result & return the value whereas assigning the `Double({ mantissa: 0 })` is less computationally intensive compared to the former & costs less gas.

```solidity
File: main/contracts/Lens/PoolLens.sol

L:466       Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });

L:486       Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L466
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L486

```solidity
File: main/contracts/Rewards/RewardsDistributor.sol

L:438       Double memory ratio = supplyTokens > 0
                ? fraction(accruedSinceUpdate, supplyTokens)
                : Double({ mantissa: 0 }); 

L:466       Double memory ratio = borrowAmount > 0
                ? fraction(accruedSinceUpdate, borrowAmount)
                : Double({ mantissa: 0 });
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#LL438C13-L440C43
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#LL466C13-L468C43

### Recommendation

```solidity
File: main/contracts/Lens/PoolLens.sol

L:466       Double memory ratio = !(borrowAmount > 0) ? Double({ mantissa: 0 }) : fraction(tokensAccrued, borrowAmount);

L:486       Double memory ratio = !(supplyTokens > 0) ? Double({ mantissa: 0 }) : fraction(tokensAccrued, supplyTokens);
```
```solidity
File: main/contracts/Rewards/RewardsDistributor.sol

L:438       Double memory ratio = !(supplyTokens > 0)
                ? Double({ mantissa: 0 }) : 
                fraction(accruedSinceUpdate, supplyTokens);

L:466       Double memory ratio = borrowAmount > 0
                ? Double({ mantissa: 0 }) :
                fraction(accruedSinceUpdate, borrowAmount);
```


