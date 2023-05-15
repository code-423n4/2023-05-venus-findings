| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable) |
| 2 | [state variables should be cached in stack variables rather than re-reading them from storage](#2-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storagethe-ones-not-publicly-known) |
| 3 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#3-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 4 | [not using the named return variables when a function returns, wastes deployment gas](#4-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 5 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#5-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 6 | [using > 0 costs more gas than != 0 when used on a uint in a require() statement](#6-using--0-costs-more-gas-than--0-when-used-on-a-uint-in-a-require-statement) |
| 7 | [use a more recent version of solidity](#7-use-the-most-recent-version-of-solidity-to-save-gas) |
| 8 | [using `calldata` instead of `memory` for read-only arguments in external functions saves gas](#8-using-calldata-instead-of-memory-for-read-only-arguments-in-external-functions-saves-gas) |
| 9 | [ Ternary over if ... else](#9-ternary-over-if--else) |
| 10 | [public functions not called by the contract should be declared external instead](#10-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 11 | [should use arguments instead of state variable](#11-should-use-arguments-instead-of-state-variable) |
| 12 | [Use assembly to check for address(0)](#12-use-assembly-to-check-for-address0) |
| 13 | [use existing cached version of state var](#13-use-existing-cached-version-of-state-var) |
| 14 | [instead of assigning to zero use `delete`](#14-instead-of-assigning-to-zero-use-delete) |
| 15 | [part of the code can be pre calculated](#15-part-of-the-code-can-be-pre-calculated) |
| 16 | [Duplicated require()/revert() checks should be refactored to a modifier or function](#16-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function) |
| 17 | [cache parts of code for second use](#17-cache-parts-of-code-for-second-use) |
| 18 | [Non-usage of specific imports](#18-non-usage-of-specific-imports) |
| 19 | [require() Should Be Used Instead Of assert()](#19-require-should-be-used-instead-of-assert) |


## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas).

- [BaseJumpRateModelV2.sol#L18](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L18)


## 2. state variables should be cached in stack variables rather than re-reading them from storage(the ones not publicly known)

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`kink` has the possibility to be read from storage 3 times. if we cache it before the #L179 check if the check passes we lose only 3 gas but if it fails we will save 197 gas.
- [BaseJumpRateModelV2.sol#L182-L185](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L182-L185)

for `poolsAssetsReserves[comptroller][asset] -= amount` to happen a storage read on `poolsAssetsReserves[comptroller][asset]` will happen but we can have it from before because its checked in the require above it as well. we can reduce one complex SLOAD if we cache it before the require #L72.

`recoveredAmount_ <= badDebt` would not revert and badDebt gonna be read from storage again later so we can cache `badDebt` before the require so we can save a highly possible 97 gas.
- [VToken.sol#L490](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L490)

usually the check (#L1215) will fail and `totalReserves` will be read from storage in both #L1215 and #L1223. so we can cache it before #L1215 to save 97 gas.
- [VToken.sol#L1215-L1223](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1215-L1223)

`protocolShareReserve` is a address that is being read from storage 3 times. we can save 200 gas by caching it before
- [VToken.sol#L1230-L1235](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1230-L1235)

`poolRegistry` is being read twice if the check in #L157 passes, which usually does. cache it before to save 97 gas
- [RiskFund.sol#L157-L170](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L157-L170)

`shortfall` is being read twice if the check in #L191 passes, which usually does. cache it before to save 97 gas
- [RiskFund.sol#L191-L194](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L191-L194)

`poolReserves[comptroller]` is being read twice if the check in #L192 passes, which usually does. cache it before to save gas
- [RiskFund.sol#L192-L193](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L192-L193)

`convertibleBaseAsset` will be read twice if `underlyingAsset != convertibleBaseAsset` condition happens so we can cache it before the check to possibly save 97 gas by only risking losing 3 gas if the check fails.
- [RiskFund.sol#L252-L255](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L252-L255)

if `snapshot.totalCollateral <= minLiquidatableCollateral` happens then revert will happen and `minLiquidatableCollateral` will be read from storage again. so we can cache it before the if check and risk losing just 3 gas but we will have a possible 97 gas save if the revert happens.
- [Comptroller.sol#L459-L461](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L459-L461)

if `snapshot.totalCollateral > minLiquidatableCollateral` happens then revert will happen and `minLiquidatableCollateral` will be read from storage again. so we can cache it before the if check and risk losing just 3 gas but we will have a possible 97 gas save if the revert happens.
- [Comptroller.sol#L591-L592](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L591-L592)
- [Comptroller.sol#L646-L648](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L646-L648)

`poolRegistry` is being read twice inside 2 different require checks which usually pass so we can cache it before and save 97 gas. if the first require fails we only lose 3 gas otherwise its 97 gas save. 
- [ReserveHelpers.sol#L53-L55](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L53-L55)


## 3. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

- [ReserveHelpers.sol#L66](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L66)
- [ReserveHelpers.sol#L67](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L67)


## 4. not using the named return variables when a function returns, wastes deployment gas

- [Comptroller.sol#L989-L991](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L989-L991)
- [Comptroller.sol#L1017-L1020](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1017-L1020)
- [Comptroller.sol#L1088](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1088)
- [Comptroller.sol#L1119](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1119)
- [Comptroller.sol#L1279](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1279)
- [Comptroller.sol#L1302](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1302)
- [Comptroller.sol#L1405-L1408](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1405-L1408)

- [VToken.sol#L566-L569](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L566-L569)

- [PoolRegistry.sol#L222](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L222)


## 5. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [ComptrollerStorage.sol#L103](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103)


## 6. using > 0 costs more gas than != 0 when used on a uint in a require() statement

This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

- [VToken.sol#L1369](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1369)

- [RiskFund.sol#L82](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L82)
- [RiskFund.sol#L83](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L83)
- [RiskFund.sol#L139](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L139)


## 7. use the most recent version of solidity to save gas

Use a solidity version of at least 0.8.15 because the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
Use a solidity version of at least 0.8.17 to  get prevention of the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.


## 8. using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

- [Comptroller.sol#L154](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154)

- [RewardsDistributor.sol#L198-L200](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L198-L200) 3 instances


## 9. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [VToken.sol#L1078-L1081](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1078-L1081)
- [VToken.sol#L1313-L1317](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1313-L1317)

- [Shortfall.sol#L241-L245](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L241-L245)

- [RewardsDistributor.sol#L222-L227](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L)


## 10. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [VToken.sol#L625](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L625)
- [VToken.sol#L645](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L645)


## 11. should use arguments instead of state variable

This will save 100 gas because it gets rid of a storage read and instead uses a argument that we already have which gives the same result

use `newCloseFactorMantissa` instead of `closeFactorMantissa` inside to save 100 gas
- [Comptroller.sol#L709](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L709)

use `initialExchangeRateMantissa_` instead of `initialExchangeRateMantissa`
- [VToken.sol#L1369](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1369)

use `underlying_` instead of `underlying`
- [VToken.sol#L1391](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1391)


## 12. Use assembly to check for address(0)

saves 6 gas per instance

- [UpgradeableBeacon.sol#L8](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L8)

- [Comptroller.sol#L128](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128)
- [Comptroller.sol#L962](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L962)

- [VToken.sol#L72](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72)
- [VToken.sol#L134](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L134)
- [VToken.sol#L196](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L196)
- [VToken.sol#L626](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L626)
- [VToken.sol#L646](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L646)
- [VToken.sol#L1399](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1399)
- [VToken.sol#L1408](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1408)

- [Shortfall.sol#L137](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L137)
- [Shortfall.sol#L138](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L138)
- [Shortfall.sol#L166](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L166)
- [Shortfall.sol#L167](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L167)
- [Shortfall.sol#L169](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L169)
- [Shortfall.sol#L170](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L170)
- [Shortfall.sol#L180](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L180)
- [Shortfall.sol#L189](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L189)
- [Shortfall.sol#L214](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L214)
- [Shortfall.sol#L349](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L349)
- [Shortfall.sol#L468](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L468)

- [PoolRegistry.sol#L225](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L225)
- [PoolRegistry.sol#L226](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L226)
- [PoolRegistry.sol#L257](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L257)
- [PoolRegistry.sol#L258](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L258)
- [PoolRegistry.sol#L259](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L259)
- [PoolRegistry.sol#L260](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L260)
- [PoolRegistry.sol#L264](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L264)
- [PoolRegistry.sol#L396](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L396)
- [PoolRegistry.sol#L422](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422)
- [PoolRegistry.sol#L431](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L431)

- [RiskFund.sol#L80](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80)
- [RiskFund.sol#L81](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L81)
- [RiskFund.sol#L100](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L100)
- [RiskFund.sol#L111](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L111)
- [RiskFund.sol#L127](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L127)
- [RiskFund.sol#L157](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L157)

- [BaseJumpRateModelV2.sol#L72](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L72)

- [ProtocolShareReserve.sol#L40](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40)
- [ProtocolShareReserve.sol#L41](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L41)
- [ProtocolShareReserve.sol#L54](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L54)
- [ProtocolShareReserve.sol#L71](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L71)

- [ReserveHelpers.sol#L40](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L40)
- [ReserveHelpers.sol#L52](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L52)
- [ReserveHelpers.sol#L53](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L53)
- [ReserveHelpers.sol#L55](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L55)


## 13. use existing cached version of state var

we already have the cached version of `rewardsDistributors.length` inside `rewardsDistributorsLength`#L930 so we dont need to read `rewardsDistributors.length` from storage again in #L940
- [Comptroller.sol#L940](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L940)

value of `rewardTokenAccrued[contributor]` is inside `contributorAccrued` as well and wee can use that instead which will be way cheaper
- [RewardsDistributor.sol#L268](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L268)

for `assetsReserves[asset] += balanceDifference` to happen a storage read on `assetsReserves[asset]` will happen but we have it from before inside `assetReserve`
- [ReserveHelpers.sol#L66](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L66)


## 14. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variable to default value. so it is encouraged if the data is no longer needed use delete instead.

- [Comptroller.sol#L812](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L812)
- [Comptroller.sol#L813](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L813)

- [VToken.sol#L424](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L424)

- [Shortfall.sol#L370](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L370)
- [Shortfall.sol#L371](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L371)


## 15. part of the code can be pre calculated

these parts of the code can be pre calculated and given to the contract as constants this will stop the use of extra operations
even if its for code readability consider putting comments instead.

`2**224` use the answer instead. otherwise every time this function is called gas is wasted calculating this
- [ExponentialNoError.sol#L64](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L64)

`2**32`
- [ExponentialNoError.sol#L69](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L69)

`expScale / 2` constants will keep the expression and not the result so every time the `halfExpScale` constant is used `expScale / 2` will be calculated so just pre calculate it.
- [ExponentialNoError.sol#L22](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22)


## 16. Duplicated require()/revert() checks should be refactored to a modifier or function

Saves deployment costs

`require(_isStarted(auction)` is used 3 times 
- [Shortfall.sol#L161](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L161)


## 17. cache parts of code for second use

cache the result of those operations in new stack variables and then use those to assign values to state variables and use those ones inside emit to stop 400 gas usage
- [BaseJumpRateModelV2.sol#L162](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L162)


## 18. Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly.
- [more here](https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files)

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

all contracts


## 19.  require() Should Be Used Instead Of assert()

- [Comptroller.sol#L225](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L225)
