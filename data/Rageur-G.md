## GAS-1: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* ReserveHelpers.sol (Line: 66, 67)


## GAS-2: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* Comptroller.sol (Line: 582)
* PoolLens.sol (Line: 460, 481)
* VToken.sol (Line: 136, 628, 648)

## GAS-3: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* BaseJumpRateModelV2.sol (Line: 137)
* Comptroller.sol (Line: 256, 333, 367, 395, 442, 464, 497, 591, 595, 646, 661, 1245, 1262)
* PoolLens.sol (Line: 462, 462, 470, 483, 483, 490)
* PoolRegistry.sol (Line: 431)
* ProtocolShareReserve.sol (Line: 54, 71)
* ReserveHelpers.sol (Line: 39, 40, 51, 52, 53)
* RewardsDistributor.sol (Line: 261, 284, 309, 435, 435, 446, 463, 463, 474)
* RiskFund.sol (Line: 80, 81, 82, 100, 127, 139, 157, 171, 191)
* Shortfall.sol (Line: 137, 161, 164, 164, 164, 164, 179, 180, 189, 212, 213, 227, 241, 278, 349, 361)
* VToken.sol (Line: 134, 395, 456, 489, 626, 646, 752, 808, 882, 939, 1035, 1045, 1107, 1156, 1183, 1205, 1248, 1408)
* WhitePaperInterestRateModel.sol (Line: 92)

## GAS-4: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* Comptroller.sol (Line: 993, 1022, 1088, 1119, 1410)
* PoolRegistry.sol (Line: 222)
* VToken.sol (Line: 571)

## GAS-5: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* BaseJumpRateModelV2.sol (Line: 112)
* Comptroller.sol (Line: 1144)
* ReserveHelpers.sol (Line: 50)
* VToken.sol (Line: 625, 645)
* WhitePaperInterestRateModel.sol (Line: 67)

## GAS-6: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* RewardsDistributor.sol (Line: 98)
* VToken.sol (Line: 33)

## GAS-7: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* BaseJumpRateModelV2.sol (Line: 162)
* MaxLoopsLimitHelper.sol (Line: 31)

## GAS-8: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* ExponentialNoError.sol (Line: 63, 65, 68, 70)
* RewardsDistributor.sol (Line: 126, 433, 461)
* VToken.sol (Line: 59, 1350)

## GAS-9: Use ```assembly``` to write address storage values

### Affected file

* BaseJumpRateModelV2.sol (Line: 74, 157, 158, 159, 160)
* Comptroller.sol (Line: 130)
* MaxLoopsLimitHelper.sol (Line: 29)
* PoolRegistry.sol (Line: 175, 176, 177, 426, 435)
* ProtocolShareReserve.sol (Line: 45, 46)
* RewardsDistributor.sol (Line: 117, 118, 128, 129, 341, 379, 431, 459)
* RiskFund.sol (Line: 88, 89, 90, 118, 129, 141)
* Shortfall.sol (Line: 144, 145, 146, 147, 148, 149, 159, 210, 276, 297, 311, 324, 337, 351, 363)
* WhitePaperInterestRateModel.sol (Line: 37, 38)

## GAS-10: Use assembly to check for address(0)

### Description

Saves 6 gas per instance if using assembly to check for address(0).

### Remediation

```
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

### Affected file

* BaseJumpRateModelV2.sol (Line: 72)
* Comptroller.sol (Line: 128, 962)
* PoolRegistry.sol (Line: 225, 226, 257, 258, 259, 260, 263, 396)
* ProtocolShareReserve.sol (Line: 40, 41, 54, 71)
* ReserveHelpers.sol (Line: 40, 52, 53, 54)
* RiskFund.sol (Line: 80, 81, 100, 111, 127, 157)
* Shortfall.sol (Line: 137, 138, 164, 164, 164, 164, 213, 349)
* UpgradeableBeacon.sol (Line: 8)
* VToken.sol (Line: 72, 134, 196, 626, 646)

## GAS-11: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* Comptroller.sol (Line: 262, 351)
* VToken.sol (Line: 1055, 1314, 1331)

## GAS-12: Use elementary types or a user-defined type instead of a struct that has only one member

### Affected file

* ExponentialNoError.sol (Line: 12, 16)

## GAS-13: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* BaseJumpRateModelV2.sol (Line: 98)
* Comptroller.sol (Line: 501, 504)
* PoolRegistry.sol (Line: 415, 416, 417)
* ReserveHelpers.sol (Line: 59)
* RewardsDistributor.sol (Line: 417)
* RiskFund.sol (Line: 264)
* Shortfall.sol (Line: 187, 193)
* VToken.sol (Line: 420, 527, 749, 842, 869, 879, 936, 1027, 1068, 1078, 1079, 1104, 1134, 1135, 1274, 1275, 1276, 1304, 1423)

## GAS-14: Use nested if and avoid multiple check combinations

### Description

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Affected file

* Comptroller.sol (Line: 755)
* PoolLens.sol (Line: 462, 483, 506, 526)
* RewardsDistributor.sol (Line: 261, 348, 386, 418, 435, 463)
* VToken.sol (Line: 837)

## GAS-15: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* Comptroller.sol (Line: 367, 1262)
* PoolLens.sol (Line: 462, 470, 483, 490, 506, 526)
* RewardsDistributor.sol (Line: 261, 418, 435, 446, 463, 474)
* RiskFund.sol (Line: 82, 83, 139, 244)
* VToken.sol (Line: 818, 837, 1369)

## GAS-16: Using Solidity version 0.8.19 will provide an overall gas optimization

### Description

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath.
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining.
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

### Affected file

* BaseJumpRateModelV2.sol (Line: 2)
* Comptroller.sol (Line: 2)
* ComptrollerInterface.sol (Line: 2)
* ComptrollerStorage.sol (Line: 2)
* ErrorReporter.sol (Line: 2)
* ExponentialNoError.sol (Line: 2)
* IPancakeswapV2Router.sol (Line: 2)
* IProtocolShareReserve.sol (Line: 2)
* IRiskFund.sol (Line: 2)
* IShortfall.sol (Line: 2)
* InterestRateModel.sol (Line: 2)
* JumpRateModelFactory.sol (Line: 2)
* JumpRateModelV2.sol (Line: 2)
* MaxLoopsLimitHelper.sol (Line: 2)
* PoolLens.sol (Line: 2)
* PoolRegistry.sol (Line: 2)
* PoolRegistryInterface.sol (Line: 2)
* ProtocolShareReserve.sol (Line: 2)
* ReserveHelpers.sol (Line: 2)
* RewardsDistributor.sol (Line: 2)
* RiskFund.sol (Line: 2)
* Shortfall.sol (Line: 2)
* UpgradeableBeacon.sol (Line: 2)
* VToken.sol (Line: 2)
* VTokenInterfaces.sol (Line: 2)
* VTokenProxyFactory.sol (Line: 2)
* WhitePaperInterestRateModel.sol (Line: 2)
* WhitePaperInterestRateModelFactory.sol (Line: 2)

## GAS-17: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* BaseJumpRateModelV2.sol (Line: 18)
* Comptroller.sol (Line: 27)
* WhitePaperInterestRateModel.sol (Line: 22, 27)