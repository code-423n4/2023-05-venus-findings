|      |                                    Issues                                    | Instances |
| :--- | :--------------------------------------------------------------------------: | --------: |
| G-01 |              Use assembly to check for `address(0)`. Save Gas.               |        32 |
| G-02 |        It is not necessary to declare. No need to read `amountLeft`.         |         1 |
| G-03 | Consider using Assembly sometimes for `Loops` and `If` statements. Save gas. |         1 |
| G-04 |           When possible, use assembly instead of `unchecked{++i}`.           |        37 |
| G-05 |             Consider use assembly for math (add, sub, mul, div).             |        42 |

## [G-01] Use assembly to check for `address(0)`. Save Gas.

There are 32 instances:

Example:

```java
function ownerNotZero(address _addr) public pure {
    require(_addr != address(0), "zero address)");
}
```

Gas: 258

```java
function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 252

[Comptroller.sol#L128](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128)

```java
require(poolRegistry_ != address(0), "invalid pool registry address");
```

[Comptroller.sol#L962](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L962)

```java
require(address(newOracle) != address(0), "invalid price oracle address");
```

[VToken.sol#L72](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72)

```java
require(admin_ != address(0), "invalid admin address");
```

[VToken.sol#L134](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L134)

```java
require(spender != address(0), "invalid spender address");
```

[VToken.sol#L196](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L196)

```java
require(minter != address(0), "invalid minter address");
```

[VToken.sol#L626](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L626)

```java
require(spender != address(0), "invalid spender address");
```

[VToken.sol#L646](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L646)

```java
require(spender != address(0), "invalid spender address");
```

[VToken.sol#L1399](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1399-L1401)

```java
if (shortfall_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

[VToken.sol#L1408-L1410](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1408-L1410)

```java
if (protocolShareReserve_ == address(0)) {
             revert ZeroAddressNotAllowed();
        }
```

[Shortfall/Shortfall.sol#L137](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L137)

```java
require(convertibleBaseAsset_ != address(0), "invalid base asset address");
```

[Shortfall/Shortfall.sol#L349](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L349)

```java
require(_poolRegistry != address(0), "invalid address");
```

[Pool/PoolRegistry.sol#L225-L226](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L225-L226)

```java
require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");
```

[Pool/PoolRegistry.sol#L257-L260](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L257-L260)

```java
require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

require(input.asset != address(0), "PoolRegistry: Invalid asset address");

require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");
```

[Pool/PoolRegistry.sol#L396](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L396)

```java
require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");
```

[Pool/PoolRegistry.sol#L431-L433](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L431-L433)

```java
if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

[RiskFund/RiskFund.sol#L80-L81](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80-L81)

```java
require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
```

[RiskFund/RiskFund.sol#L100](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L100)

```java
require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
```

[RiskFund/RiskFund.sol#L111](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L111)

```java
require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");
```

[RiskFund/RiskFund.sol#L127](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L127)

```java
require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");
```

[RiskFund/RiskFund.sol#L157](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L157)

```java
require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
```

[RiskFund/ProtocolShareReserve.sol#L40-L41](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40-L41)

```java
require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");
```

[RiskFund/ProtocolShareReserve.sol#L54](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L54)

```java
require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");
```

[RiskFund/ProtocolShareReserve.sol#L71](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L71)

```java
require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
```

[RiskFund/ReserveHelpers.sol#L52-L53](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L52-L53)

```java
require(asset != address(0), "ReserveHelpers: Asset address invalid");

require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
```

[Proxy/UpgradeableBeacon.sol#L8](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L8)

```java
require(implementation_ != address(0), "Invalid implementation address");
```

Recommended Mitigations Steps:

-   Use Assembly for this check

## [G-02] It is not necessary to declare. No need to read `amountLeft`.

The an instance:

[Rewards/RewardsDistributor.sol#L182-L183](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L182-L183)

```java
uint256 amountLeft = _grantRewardToken(recipient, amount);
require(amountLeft == 0, "insufficient rewardToken for grant");
```

Recommended Mitigations Steps:

change to:

```ts
require(_grantRewardToken(recipient, amount) ==
    0, 'insufficient rewardToken for grant');
```

## [G-03] Consider using Assembly sometimes for `Loops` and `If` statements. Save gas.

There an instance:

Example:

```java
function howManyEvens(uint256 startNum, uint256 endNum) external pure returns(uint256) {
    // the value we will return
    uint256 ans;
    assembly {
        // syntax for for loop
        for { let i := startNum } lt( i, add(endNum, 1)  ) { i := add(i,1) }
        {
            // if i == 0 skip this iteration
            if iszero(i) {
                continue
            }
            // checks if i % 2 == 0
            // we could of used iszero, but I wanted to show you eq()
            if  eq( mod( i, 2 ), 0 ) {
                ans := add(ans, 1)
            }
        }
    }
    return ans;
}
```

[Comptroller.sol#L216-L222](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L216-L222)

```java
uint256 assetIndex = len;
for (uint256 i; i < len; ++i) {
    if (userAssetList[i] == vToken) {
        assetIndex = i;
        break;
    }
}
```

Recommended Mitigations Steps:

-   See the example

## [G-04] When possible, use assembly instead of `unchecked{++i}`.

_[You can also use `unchecked{++i;}`](https://gist.github.com/CloudEllie/6639dbfd7dc1809a3baa28bb2895e1d9#g10-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for--and-while-loops) for even more gas savings but this will not check to see if `i` overflows. For best gas savings, use inline assembly, however this limits the functionality you can achieve._

Example:

```java
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```

Gas: 1329

```java
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```

Gas: 709

Recommended Mitigations Steps:

-   See the example. Implement whenever possible

## [G-05] Consider use assembly for math (add, sub, mul, div).

_Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety._

There are 42 instances:

Example Addition:

```ts
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```

Gas: 303

```ts
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 263

[ExponentialNoError.sol#L81-L83](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L81-L83)

```ts
function add_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }
```

Example Subtraction:

```ts
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```

Gas: 300

```ts
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 263

[ExponentialNoError.sol#L93-L95](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L93-L95)

```ts
function sub_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a - b;
    }
```

Example Multiplication:

```ts
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```ts
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

[ExponentialNoError.sol#L121-L123](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L121-L123)

```ts
function mul_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a * b;
    }
```

Example Division:

```ts
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```

Gas: 325

```ts
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Gas: 265

[ExponentialNoError.sol#L149-L151](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L149-L151)

```ts
function div_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a / b;
    }
```

[BaseJumpRateModelV2.sol#L118](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L118)

```ts
uint256 oneMinusReserveFactor = BASE - reserveFactorMantissa;
```

[BaseJumpRateModelV2.sol#L120](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L120)

```ts
uint256 rateToPool = (borrowRate * oneMinusReserveFactor) / BASE;
```

[BaseJumpRateModelV2.sol#L141](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L141)

```ts
return (borrows * BASE) / (cash + borrows - reserves);
```

[BaseJumpRateModelV2.sol#L157-L159](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L157-L159)

```ts
baseRatePerBlock = baseRatePerYear / blocksPerYear;
multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
```

[BaseJumpRateModelV2.sol#L180](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L180)

```ts
return (util * multiplierPerBlock) / BASE + baseRatePerBlock;
```

[BaseJumpRateModelV2.sol#L182](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L182)

```ts
uint256 normalRate = ((kink * multiplierPerBlock) / BASE) + baseRatePerBlock;
```

[BaseJumpRateModelV2.sol#L185](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L185)

```ts
excessUtil = util - kink;
```

[BaseJumpRateModelV2.sol#L187](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L187)

```ts
return (excessUtil * jumpMultiplierPerBlock) / BASE + normalRate;
```

[WhitePaperInterestRateModel.sol#L37-L38](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L37-L38)

```ts
baseRatePerBlock = baseRatePerYear / blocksPerYear;
multiplierPerBlock = multiplierPerYear / blocksPerYear;
```

[WhitePaperInterestRateModel.sol#L56](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L56)

```ts
return (ur * multiplierPerBlock) / BASE + baseRatePerBlock;
```

[WhitePaperInterestRateModel.sol#L73](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L73)

```ts
uint256 oneMinusReserveFactor = BASE - reserveFactorMantissa;
```

[WhitePaperInterestRateModel.sol#L75](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L75)

```ts
uint256 rateToPool = (borrowRate * oneMinusReserveFactor) / BASE;
```

[WhitePaperInterestRateModel.sol#L96](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L96)

```ts
return (borrows * BASE) / (cash + borrows - reserves);
```

[RiskFund/ReserveHelpers.sol#L64](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L64)

```ts
balanceDifference = currentBalance - assetReserve;
```

[Shortfall/Shortfall.sol#L244](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L244)

```ts
riskFundBidAmount = (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS;
```

[VToken.sol#L784](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L784)

```ts
totalSupply = totalSupply + mintTokens;
```

[VToken.sol#L857](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L857)

```ts
totalSupply = totalSupply - redeemTokens;
```

[VToken.sol#L897-L898](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L897-L898)

```ts
uint256 accountBorrowsNew = accountBorrowsPrev + borrowAmount;
uint256 totalBorrowsNew = totalBorrows + borrowAmount;
```

[VToken.sol#L965-L966](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L965-L966)

```ts
uint256 accountBorrowsNew = accountBorrowsPrev - actualRepayAmount;
uint256 totalBorrowsNew = totalBorrows - actualRepayAmount;
```

[VToken.sol#L1117](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1117)

```ts
uint256 liquidatorSeizeTokens = seizeTokens - protocolSeizeTokens;
```

[VToken.sol#L1120](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1120)

```ts
uint256 totalReservesNew = totalReserves + protocolSeizeAmount;
```

[VToken.sol#L1128](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1128)

```ts
totalSupply = totalSupply - protocolSeizeTokens;
```

[VToken.sol#L1188](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1188)

```ts
totalReservesNew = totalReserves + actualAddAmount;
```

[VToken.sol#L1278](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1278)

```ts
return balanceAfter - balanceBefore;
```

[VToken.sol#L1320](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1320)

```ts
uint256 allowanceNew = startingAllowance - tokens;
```

[VToken.sol#L1453-L1455](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1453-L1455)

```ts
uint256 principalTimesIndex = borrowSnapshot.principal * borrowIndex;

        return principalTimesIndex / borrowSnapshot.interestIndex;
```

[VToken.sol#L1477-L1478](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1477-L1478)

```ts
uint256 cashPlusBorrowsMinusReserves = totalCash + totalBorrows + badDebt - totalReserves;
uint256 exchangeRate = (cashPlusBorrowsMinusReserves * expScale) / _totalSupply;
```

[Comptroller.sol#L353](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L353)

```ts
uint256 nextTotalBorrows = totalBorrows + borrowAmount;
```

[Comptroller.sol#L1348](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1348)

```ts
uint256 borrowPlusEffects = snapshot.borrows + snapshot.effects;
```

[Comptroller.sol#L1352](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1352)

```ts
snapshot.liquidity = snapshot.weightedCollateral - borrowPlusEffects;
```

[Comptroller.sol#L1356](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1356)

```ts
snapshot.shortfall = borrowPlusEffects - snapshot.weightedCollateral;
```

Recommended Mitigations Steps:

-   See the example above
