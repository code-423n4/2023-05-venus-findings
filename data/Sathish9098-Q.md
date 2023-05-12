## LOW FINDINGS 

| Issue Count | Issues | Instances |
|-----------------|-----------------|-----------------|
| [L-1] | MIXING AND OUTDATED COMPILER   |  4 |
| [L-2] | Sanity/Threshold/Limit Checks   | 10  |
| [L-3] | Unsafe typecasting method is used to cast bytes32 to uint256   | 5  |
| [L-4] | Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256    | 13  |
| [L-5] | Prevent division by 0  | 3  |
| [L-6] | Function may run out of gas  | -  |
| [L-7] |  Array lengths not checked before batch operations | 1  |
| [L-8] | Missing event and or timelock for critical parameter change | -  |
| [L-9] |  Don’t use payable.call()  |  1 |
| [L-10] | Inconsistent spacing in comments  | -  |
| [L-11] | A single point of failure   | 12  |
| [L-12] | Loss of precision due to rounding   | 5  |
| [L-13] | Low-level calls that are unnecessary for the system should be avoided   |  1 |
| [L-14] | Project Upgrade and Stop Scenario should be  | -  |
| [L-15] |  Update codes to avoid Compile Errors  | 3  |

# NON CRITICAL FINDINGS

| Issue Count | Issues | Instances |
|-----------------|-----------------|-----------------|
| [NC-1]  | Named imports can be used  |  3 |
| [NC-2]  | Remove commented out code  |  - |
| [NC-3]  |  Test environment comments and codes should not be in the main version | 1  |
| [NC-4]  | Add a timelock to critical functions  |  - |
| [NC-5]  | No same value control  |  - |
| [NC-6]  | Critical changes should use two-step procedure  | -  |
| [NC-7]  | Emit both old and new values in critical changes  |  - |
| [NC-8]  | Missing NATSPEC  |  - |
| [NC-9]  | For functions, follow Solidity standard naming conventions (internal function style rule)   |  14 |
| [NC-10]  | FUNCTIONS,PARAMETERS,MODIFIERS AND VARIABLES IN SNAKE CASE  | -  |
| [NC-11]  | Use a more recent version of solidity  | -  |
| [NC-12]  | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS  | -  |
| [NC-13]  | NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING  | -  |
| [NC-14]  | Mark visibility of initialize(…) functions as external   | 1  |
| [NC-15]  |  Contract layout and order of functions |  - |
| [NC-16]  |  Pragma float | -  |
| [NC-17]  | Use solidity naming conventions for state variables   |  6 |
| [NC-18]  | Interchangeable usage of uint and uint256  |  - |
| [NC-19]  | TYPOS  | 3  |
| [NC-20]  | public functions not called by the contract should be declared external instead   |  6 |
| [NC-21]  | Use scientific notations rather than exponential notations  |  5 |
| [NC-22]  |  Use underscores for number literals  |  7 |
| [NC-23]  | Unused variables   |  2 |
| [NC-24]  | Keccak Constant values should used to immutable rather than constant   |  1 |
| [NC-25]  | Tokens accidentally sent to the contract cannot be recovered  | -  |
| [NC-26]  | Use SMTChecker  | -  |
| [NC-27]  | Constants on the left are better  | -  |
| [NC-28]  | Use constants instead of using numbers directly     |  5 |
| [NC-29]  |  According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword   |   6|
| [NC-30]  | Assembly Codes Specific – Should Have Comments   | 2  |
| [NC-31]  | Large multiples of ten should use scientific notation (e.g. 1e5) rather than decimal literals (e.g. 100000), for readability    |  4 |

##



## [L-1] MIXING AND OUTDATED COMPILER

### Impact 
Mixing pragma is not recommended. Because different compiler versions have different meanings and behaviors, it also significantly raises maintenance costs. As a result, depending on the compiler version selected for any given file, deployed contracts may have security issues

The minimum required version must be 0.8.19 otherwise, contracts will be affected by the following important bug fixes

- 0.8.14:
ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.
- 0.8.15
Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.
- 0.8.16
Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.
- 0.8.17
Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.
Apart from these, there are several minor bug fixes and improvements


## Recommended Mitigation:
Use More recent version of solidity 

##

## [L-2] Lack of integer validations before executions


## [L-3] safeApprove already depreacted in openzheplin contracts 



## [L-3] Unsafe typecasting method is used to cast bytes32 to uint256 

To convert a bytes32 variable to a uint256 variable in Solidity, you can use the uint256 cast with the abi.decode function

### EXAMPLE :

```solidity
bytes32 id_ = 0x1234567890123456789012345678901234567890123456789012345678901234;
uint256 id = uint256(abi.decode(bytes(id_), (uint256)));
```
```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

> id_ is type casted from unsafe way from bytes32 to uint256

297: function bump(bytes32 id_) external can_buy(uint256(id_)) {
298: uint256 id = uint256(id_);

565: require(buy(uint256(id), maxTakeAmount));

753: require(buy(uint256(id), maxTakeAmount));

757: require(cancel(uint256(id)));

```

[RubiconMarket.sol#L565](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L565),[RubiconMarket.sol#L753](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L753)

### Recommended Mitigation

Use safe casting way instead incompatible way 

##

## [L-7] Array lengths not checked before batch operations

The batchRequote function provided takes in several arrays as input parameters. It is important to note that if the lengths of these arrays do not match, the function may behave unexpectedly or throw an error 

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

   function batchRequote(
        uint[] calldata ids,
        uint[] calldata payAmts,
        address[] calldata payGems,
        uint[] calldata buyAmts,
        address[] calldata buyGems
    ) external {
        for (uint i = 0; i < ids.length; i++) {
            cancel(ids[i]);
            this.offer(
                payAmts[i],
                ERC20(payGems[i]),
                buyAmts[i],
                ERC20(buyGems[i])
            );
        }
    }


```

[RubiconMarket.sol#L917-L933](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L917-L933)


Recommended Mitigation:

require(
            ids.length == payAmts.length &&
                ids.length == payGems.length &&
                ids.length == buyAmts.length && ids.length == buyGems.length
            "Array lengths do not match"
        );

##

## [L-8] Missing events for critical parameter changes

Events help non-contract tools to track changes, and events prevent users from being surprised by changes

##

## [L-9] Don’t use payable.call()

payable.call() is a low-level method that can be used to send ether to a contract, but it has some limitations and risks as you've pointed out. One of the primary risks of using payable.call() is that it doesn't guarantee that the contract's payable function will be called successfully. This can lead to funds being lost or stuck in the contract

- The contract does not have a payable callback
- The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)
- The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin’s Address.sendValue() instead

```solidity
FILE: 2023-04-rubicon/contracts/utilities/FeeWrapper.sol

118: (bool OK, ) = payable(_feeTo).call{value: _feeAmount}("");

```
[FeeWrapper.sol#L118](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/FeeWrapper.sol#L118)

##

## [L-11] A single point of failure

Impact
The onlyOwner or auth roles has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

> onlyOwner and auth functions 

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

25:   function setOwner(address owner_) external auth {
1466: function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
1471: function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
1476: function setFeeTo(address newFeeTo) external auth returns (bool) {

```
[RubiconMarket.sol#L25](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25),[RubiconMarket.sol#L1466](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1466)

(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L955-L958)

```solidity
FILE: 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

   function buyAllAmountWithLeverage(
        address quote,
        address asset,
        uint256 quotePayAmount,
        uint256 leverage
    ) external onlyOwner {

function sellAllAmountWithLeverage(
        address asset,
        address quote,
        uint256 assetPayAmount,
        uint256 leverage
    ) external onlyOwner {

210: function closePosition(uint256 posId) external onlyOwner {
226: function increaseMargin(uint256 posId, uint256 amount) external onlyOwner {
242: function withdraw(address token, uint256 amount) external onlyOwner {

```
[Position.sol#L93-L98](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L93-L98),[Position.sol#L107-L112](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L107-L112),[Position.sol#L210](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L210),[Position.sol#L226](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L226),[Position.sol#L242](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L242)

```solidity
FILE: 2023-04-rubicon/contracts/periphery/BathBuddy.sol

function getReward(
        IERC20 rewardsToken,
        address holderRecipient
    )
        external
        override
        nonReentrant
        whenNotPaused
        updateReward(holderRecipient, address(rewardsToken))
        onlyBathHouse

function notifyRewardAmount(
        uint256 reward,
        IERC20 rewardsToken
    ) external onlyOwner updateReward(address(0), address(rewardsToken)) {

function setRewardsDuration(
        uint256 _rewardsDuration,
        address token
    ) external onlyOwner {

```
[BathBuddy.sol#L168-L177](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/periphery/BathBuddy.sol#L168-L177)

##

##

## [L-13] Low-level calls that are unnecessary for the system should be avoided

Low-level calls that are unnecessary for the system should be avoided whenever possible because low-level calls behave differently from a contract-type call. For example;

address.call(abi.encodeWithSelector("fancy(bytes32)", mybytes))`` does not verify that a target is actually a contract, while ContractInterface(address).fancy(mybytes) does.

Additionally, when calling out to functions declared view/pure, the solidity compiler would actually perform a staticcall providing additional security guarantees while a low-level call does not. Similarly, return values have to be decoded manually when performing low-level calls.

Note: if a low-level call needs to be performed, consider relying on Contract.function.selector instead of encoding using a hardcoded ABI string

```solidity
FILE: 2023-04-rubicon/contracts/utilities/FeeWrapper.sol

118: (bool OK, ) = payable(_feeTo).call{value: _feeAmount}("");

```
[FeeWrapper.sol#L118](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/FeeWrapper.sol#L118)

##

## [L-14] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN 

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-15] Update codes to avoid Compile Errors

```solidity

warning[2018]: Warning: Function state mutability can be restricted to view
   --> contracts/RubiconMarket.sol:297:5:
    |
297 |     function bump(bytes32 id_) external can_buy(uint256(id_)) {
    |     ^ (Relevant source part starts here and spans across multiple lines).



warning[2018]: Warning: Function state mutability can be restricted to pure
  --> contracts/utilities/FeeWrapper.sol:28:5:
   |
28 |     function calculateFee(
   |     ^ (Relevant source part starts here and spans across multiple lines).



warning[2018]: Warning: Function state mutability can be restricted to view
   --> contracts/utilities/poolsUtility/Position.sol:526:5:
    |
526 |     function _borrowLimit(
    |     ^ (Relevant source part starts here and spans across multiple lines).

```



##

## NON CRITICAL FINDINGS

##

## [NC-1] Named imports can be used

#### CONTEXT

ALL CONTRACTS 

It’s possible to name the imports to improve code readability. 

E.g. import "@openzeppelin/contracts/token/ERC20/IERC20.sol"; can be rewritten as import {IERC20} from “import “@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol”;

> Example 

FILE : 2023-04-rubicon/contracts/BathHouseV2.sol

```solidity 
FILE : 2023-04-rubicon/contracts/BathHouseV2.sol

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "./compound-v2-fork/InterestRateModel.sol";
import "./compound-v2-fork/CErc20Delegator.sol";
import "./compound-v2-fork/Comptroller.sol";
import "./compound-v2-fork/Unitroller.sol";
import "./periphery/BathBuddy.sol";
```
[BathHouseV2.sol#L4-L9](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/BathHouseV2.sol#L4-L9)

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/utils/StorageSlot.sol";

```
[RubiconMarket.sol#L11-L12](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L11-L12)

```solidity
FILE: 2023-04-rubicon/contracts/V2Migrator.sol

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "./compound-v2-fork/CTokenInterfaces.sol";

```
[V2Migrator.sol#L4-L6](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/V2Migrator.sol#L4-L6)

```solidity
FILE: 2023-04-rubicon/contracts/periphery/BathBuddy.sol

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

```
[BathBuddy.sol#L4-L7](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/periphery/BathBuddy.sol#L4-L7)

```solidity
FILE: 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "../../compound-v2-fork/Comptroller.sol";
import "../../compound-v2-fork/PriceOracle.sol";
import "../../BathHouseV2.sol";
import "../../RubiconMarket.sol";

```
[Position.sol#L4-L11](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L4-L11)

(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/FeeWrapper.sol#L4-L5)

### Recommended Mitigation

Name the imports to all contract scopes to improve code readability

##

## [NC-2] Remove commented out code

CONTEXT
[RubiconMarket.sol](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol)

it's generally considered good practice to remove commented out code from a codebase once it's determined that it's no longer needed or relevant. This can help keep the codebase clean, readable, and maintainable, and can help prevent confusion or errors in the future

##

## [NC-3] Test environment comments and codes should not be in the main version

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

5: // import "hardhat/console.sol";

```
##

## [NC-4] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

function setOwner(address owner_) external auth {
        owner = owner_;
        emit LogSetOwner(owner);
}

```
[RubiconMarket.sol#L25-L28](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25-L28)

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

  function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
        feeBPS = _newFeeBPS;
        return true;
    }

    function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
        StorageSlot.getUint256Slot(MAKER_FEE_SLOT).value = _newMakerFee;
        return true;
    }

    function setFeeTo(address newFeeTo) external auth returns (bool) {
        require(newFeeTo != address(0));
        feeTo = newFeeTo;
        return true;
    }

```
[RubiconMarket.sol#L1466-L1480](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1466-L1480)

##

## [NC-5] No same value control 

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

function setOwner(address owner_) external auth {
        owner = owner_;
        emit LogSetOwner(owner);
}

```
[RubiconMarket.sol#L25-L28](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25-L28)

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

  function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool) {
        feeBPS = _newFeeBPS;
        return true;
    }

    function setMakerFee(uint256 _newMakerFee) external auth returns (bool) {
        StorageSlot.getUint256Slot(MAKER_FEE_SLOT).value = _newMakerFee;
        return true;
    }

    function setFeeTo(address newFeeTo) external auth returns (bool) {
        require(newFeeTo != address(0));
        feeTo = newFeeTo;
        return true;
    }

```
[RubiconMarket.sol#L1466-L1480](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1466-L1480)

##

## [NC-6] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

function setOwner(address owner_) external auth {
        owner = owner_;
        emit LogSetOwner(owner);
}

```
[RubiconMarket.sol#L25-L28](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25-L28)



##

## [NC-7] Emit both old and new values in critical changes 

Emitting old and new values when critical changes are made can help you improve the reliability, accuracy, and maintainability of your systems

[RubiconMarket.sol#L25-L28](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25-L28)

##

## [NC-8] Missing NATSPEC

Consider adding NATSPEC on all public/external functions to improve documentation

[RubiconMarket.sol#L25-L27](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L25-L27)
[RubiconMarket.sol#L288-L293](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L288-L293)
[RubiconMarket.sol#L1028-L1034](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1028-L1034)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L760-L766)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L780-L787)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L801-L810)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L824-L833)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L885-L892)

##

## [NC-9] For functions, follow Solidity standard naming conventions (internal function style rule)

Description
The bellow codes don’t follow Solidity’s standard naming convention,

internal and private functions and variables : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

```solidity
FILE : FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

35: function isAuthorized(address src) internal view returns (bool) {
46: function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
50: function sub(uint256 x, uint256 y) internal pure returns (uint256 z) {
54: function mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
58: function min(uint256 x, uint256 y) internal pure returns (uint256 z) {
62: function max(uint256 x, uint256 y) internal pure returns (uint256 z) {
66: function imin(int256 x, int256 y) internal pure returns (int256 z) {
70: function imax(int256 x, int256 y) internal pure returns (int256 z) {
77: function wmul(uint256 x, uint256 y) internal pure returns (uint256 z) {
81: function rmul(uint256 x, uint256 y) internal pure returns (uint256 z) {
85: function wdiv(uint256 x, uint256 y) internal pure returns (uint256 z) {
89: function rdiv(uint256 x, uint256 y) internal pure returns (uint256 z) {

227:  uint256 internal feeBPS;
230:  address internal feeTo;
```
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L125-L130)

##

## [NC-10] FUNCTIONS,PARAMETERS,MODIFIERS AND VARIABLES IN SNAKE CASE

Use camel case for all functions, parameters and variables and snake case for constants

Snake case examples
The following variable names follow the snake case naming convention:

this_is_snake_case
build_docker_image

Camel case examples
The following are examples of variables that use the camel case naming convention:

FileNotFoundException
toString

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

219: uint256 public last_offer_id;
245: modifier can_buy(uint256 id) virtual {
251: modifier can_cancel(uint256 id) virtual {
719: modifier can_cancel(uint256 id) override {
```

##

## [NC-11] Use a more recent version of solidity

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

2: pragma solidity ^0.8.9;
```
```solidity
FILE: 2023-04-rubicon/contracts/utilities/FeeWrapper.sol

2: pragma solidity 0.8.17;
```
```solidity
FILE: 2023-04-rubicon/contracts/periphery/BathBuddy.sol

2: pragma solidity ^0.8.0;
```

##

## [NC-12] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

### Recommendation
NatSpec comments should be increased in Contracts

##

## [NC-13] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L276)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L280)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L284)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L452-L454)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L491-L496)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L511-L518)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L578-L580)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L620-L622)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1028-L1034)

##

## [NC-14] Mark visibility of initialize(…) functions as external

Description
External instead of public would give more the sense of the initialize(…) functions to behave like a constructor (only called on deployment, so should only be called externally).

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public (= “opening it up”) ✅
but it is not possible to override a function from public to external (= “narrow it down”). ❌

For above reasons you can change initialize(…) to external

(https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750)

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

700:  function initialize(address _feeTo) public {

```
[RubiconMarket.sol#L700](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L700)

##

## [NC-15] Contract layout and order of functions

The Solidity style guide recommends declaring state variables before all functions. 

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout)

CONTEXT
ALL CONTRACT

### Layout contract elements in the following order:

- Pragma statements

- Import statements

Interfaces

- Libraries

- Contracts

### Inside each contract, library or interface, use the following order:

- Type declarations

- State variables

- Events

- Modifiers

- Functions

### Recommendations 

All contracts should follow the solidity style guide 

##

## [NC-16] Pragma float

[RubiconMarket.sol](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol), [BathBuddy.sol](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol) both contracts using the floating pragma 

### Recommendation
Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package

##

## [NC-17] Use solidity naming conventions for state variables 

State variables name not starting with "_". The "_name" is reserved for internal/private functions and variables 

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html)

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

691: mapping(uint256 => sortInfo) public _rank; //doubly linked lists of sorted offer ids
692: mapping(address => mapping(address => uint256)) public _best; //id of the highest offer for a token pair
693: mapping(address => mapping(address => uint256)) public _span; //number of offers stored for token pair in sorted orderbook
694: mapping(address => uint256) public _dust; //minimum sell amount for a token to avoid dust offers
695: mapping(uint256 => uint256) public _near; //next unsorted offer id
696: uint256 public _head; //first unsorted offer id

```
[RubiconMarket.sol#L691-L696](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L691-L696)

##

## [NC-18] Interchangeable usage of uint and uint256

Consider using only one approach throughout the codebase, e.g. only uint or only uint256

(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L781-L785)
(https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L918-L921)


##

## [NC-19] TYPOS

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

/// @audit multuple => multiple
886:  /// @notice Batch offer functionality - multuple offers in a single transaction

/// @audit acumulator=> accumulator
1051: fill_amt = add(fill_amt, offers[offerId].pay_amt); //Add amount bought to acumulator

/// @audit acumulator=> accumulator
1060: fill_amt = add(fill_amt, offers[offerId].pay_amt); //Add amount bought to acumulator

/// @audit acumulator=> accumulator
1091: fill_amt = add(fill_amt, offers[offerId].pay_amt); //Add amount bought to acumulator

```
## [NC-20] public functions not called by the contract should be declared external instead

Contracts are [allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents’ functions and change the visibility from external to public.

```solidity
FILE: 2023-04-rubicon/contracts/BathHouseV2.sol

45: function getBathTokenFromAsset(
        address asset
    ) public view returns (address) {

```
[BathHouseV2.sol#L45-L47](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/BathHouseV2.sol#L45-L47)

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

574:  function getFeeBPS() public view returns (uint256) {
1010: function getFirstUnsortedOffer() public view returns (uint256) {
1016: function getNextUnsortedOffer(uint256 id) public view returns (uint256) {
1020: function isOfferSorted(uint256 id) public view returns (bool) {

1165: function getPayAmountWithFee(
        ERC20 pay_gem,
        ERC20 buy_gem,
        uint256 buy_amt
    ) public view returns (uint256 fill_amt) {


```
[RubiconMarket.sol#L574](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L574)

##

## [NC-21] Use scientific notations rather than exponential notations

https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1175-L1177
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1142-L1144
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1099-L1101
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1057-L1059
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L331
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L317

##

## [NC-22] Use underscores for number literals

```solidity
FILE: 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

459: uint256 _fee = _maxFill.mul(rubiconMarket.getFeeBPS()).div(10000);
481: uint256 _fee = _minFill.mul(_feeBPS).div(10000);
490:  _fee = _payAmount.mul(_feeBPS).div(10000);

```
```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

346:  uint256 mFee = mul(spend, makerFee()) / 100_000;
338:  uint256 fee = mul(spend, feeBPS) / 100_000;

583:  _amount -= mul(amount, feeBPS) / 100_000;
583: _amount -= mul(amount, makerFee()) / 100_000;

```
[RubiconMarket.sol#L583](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L583)


### Recommended Mitigation

```solidity
459: uint256 _fee = _maxFill.mul(rubiconMarket.getFeeBPS()).div(10_000);

```

##

## [NC-23] Unused variables 

```solidity

warning[5667]: Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/utilities/poolsUtility/Position.sol:528:9:
    |
528 |         address _asset,
    |         ^^^^^^^^^^^^^^

warning[2072]: Warning: Unused local variable.
   --> contracts/RubiconMarket.sol:298:9:
    |
298 |         uint256 id = uint256(id_);
    |         ^^^^^^^^^^


```

##

## [NC-24] Keccak Constant values should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

232: bytes32 internal constant MAKER_FEE_SLOT = keccak256("WOB_MAKER_FEE");

```
[RubiconMarket.sol#L232](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L232)

##

## [Nc-25] Tokens accidentally sent to the contract cannot be recovered

### Context
contracts/staking/NeoTokyoStaker.sol:

It can’t be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps
Add this code:

 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

##

## [NC-26] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##

## [NC-27] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns).

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1040
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1080
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1233
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1244
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1252
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1319
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1372
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1392
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1426
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L392-L393
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L340

##

## [NC-28] Use constants instead of using numbers directly  

https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L490
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L481
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L459
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L346
https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L338

##

## [NC-29] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

691: mapping(uint256 => sortInfo) public _rank; 
692: mapping(address => mapping(address => uint256)) public _best; 
693: mapping(address => mapping(address => uint256)) public _span;  sorted orderbook
694: mapping(address => uint256) public _dust; 
695: mapping(uint256 => uint256) public _near; 
```
### Recommended Mitigation

```solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

691: mapping (uint256 => sortInfo) public _rank; 
```

##

## [NC-30] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```Solidity
FILE: 2023-04-rubicon/contracts/RubiconMarket.sol

648:  assembly {
```
[RubiconMarket.sol#L648](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L648)

```solidity
FILE: 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

367:  assembly {

```
##

## [NC-31] Large multiples of ten should use scientific notation (e.g. 1e5) rather than decimal literals (e.g. 100000), for readability

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

346:  uint256 mFee = mul(spend, makerFee()) / 100_000;
338:  uint256 fee = mul(spend, feeBPS) / 100_000;

583:  _amount -= mul(amount, feeBPS) / 100_000;
583: _amount -= mul(amount, makerFee()) / 100_000;

```
[RubiconMarket.sol#L583](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L583)



























[L‑01]	Division by zero not prevented	1
[L‑02]	Loss of precision	14
[L‑03]	require() should be used instead of assert()	1
[L‑04]	safeApprove() is deprecated	4
[L‑05]	Missing checks for address(0x0) when assigning values to address state variables	1
[L‑06]	Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions	8
[L‑07]	External calls in an un-bounded for-loop may result in a DOS	4

[N‑01]	Consider disabling renounceOwnership()	7
[N‑02]	Events are missing sender information	12
[N‑03]	Variables need not be initialized to zero	2
[N‑04]	Imports could be organized more systematically	4
[N‑05]	Large numeric literals should use underscores for readability	2
[N‑06]	Constants in comparisons should appear on the left side	45
[N‑07]	Variable names don't follow the Solidity style guide	17
[N‑08]	else-block not required	2
[N‑09]	Events may be emitted out of order due to reentrancy	5
[N‑10]	if-statement can be converted to a ternary	1
[N‑11]	Consider using named mappings	33
[N‑12]	Import declarations should import specific identifiers, rather than the whole file	96
[N‑13]	Adding a return statement when the function defines a named return variable, is redundant	2
[N‑14]	public functions not called by the contract should be declared external instead	2
[N‑15]	2**<n> - 1 should be re-written as type(uint<n>).max	2
[N‑16]	constants should be defined rather than using magic numbers	11
[N‑17]	Events that mark critical parameter changes should contain both the old and the new value	5
[N‑18]	Constant redefined elsewhere	1
[N‑19]	Inconsistent spacing in comments	3
[N‑20]	Lines are too long	22
[N‑21]	Typos	10
[N‑22]	File is missing NatSpec	10
[N‑23]	NatSpec @param is missing	47
[N‑24]	NatSpec @return argument is missing	27
[N‑25]	Event is not properly indexed	12
[N‑26]	Not using the named return variables anywhere in the function is confusing	14
[N‑27]	Duplicated require()/revert() checks should be refactored to a modifier or function	5
[N‑28]	Interfaces should be indicated with an I prefix in the contract name	3
[N‑29]	Numeric values having to do with time should use time units for readability	2
[N‑30]	Consider using delete rather than assigning zero/false to clear values	8
[N‑31]	Contracts should have full test coverage	1
[N‑32]	Large or complicated code bases should implement invariant tests	1
[N‑33]	internal functions not called by the contract should be removed	2