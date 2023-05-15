## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | <x> += <y> costs more gas than <x> = <x> + <y> for state variables | 8 |
| [G&#x2011;02] | Use nested if and, avoid multiple check combinations | 11 |

### [G&#x2011;01]  <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).
Using the addition operator instead of plus-equals saves 113 gas. [Link to reference](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 8 instances of this issue.

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

74        assetsReserves[asset] -= amount;
75        poolsAssetsReserves[comptroller][asset] -= amount;
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L74-L75)

```solidity
File: contracts/RiskFund/ReserveHelpers.sol

66            assetsReserves[asset] += balanceDifference;
67            poolsAssetsReserves[comptroller][asset] += balanceDifference;
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#LL66C1-L67C74)

```solidity
File: contracts/RiskFund/RiskFund.sol

249                assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
250                poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL249C1-L250C95)

```solidity
File: contracts/VToken.sol

630        newAllowance += addedValue;
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L630)

```solidity
File: contracts/VToken.sol

652            currentAllowance -= subtractedValue;
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L652)

### Recommended Mitigation steps
Below is the recommendation should be applied on all instances,

For example:

```solidity

-        newAllowance += addedValue;
+        newAllowance = newAllowance + addedValue;
```

```solidity

-            currentAllowance -= subtractedValue;
+            currentAllowance = currentAllowance - subtractedValue;
```

### [G&#x2011;02]  Use nested if and, avoid multiple check combinations
Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 11 instances of this issue:

```solidity
File: contracts/Lens/PoolLens.sol

462        if (deltaBlocks > 0 && borrowSpeed > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL462C50-L462C50)

```solidity
File: contracts/Lens/PoolLens.sol

483        if (deltaBlocks > 0 && supplySpeed > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL483C50-L483C50)

```solidity
File: contracts/Lens/PoolLens.sol

506        if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL506C71-L506C71)

```solidity
File: contracts/Rewards/RewardsDistributor.sol

261        if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L261)

```solidity
File: contracts/Rewards/RewardsDistributor.sol

386        if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L386)

```solidity
File: contracts/Rewards/RewardsDistributor.sol

418        if (amount > 0 && amount <= rewardTokenRemaining) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L418)

```solidity
File: contracts/Rewards/RewardsDistributor.sol

435        if (deltaBlocks > 0 && supplySpeed > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L435)

```solidity
File: contracts/Rewards/RewardsDistributor.sol

463        if (deltaBlocks > 0 && borrowSpeed > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L463)

```solidity
File: contracts/Comptroller.sol

755        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L755)

```solidity
File: contracts/VToken.sol

837        if (redeemTokens == 0 && redeemAmount > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L837)

```solidity
File: contracts/VToken.sol

837        if (redeemTokens == 0 && redeemAmount > 0) {
```
[Link to code](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L837)
