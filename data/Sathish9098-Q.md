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

## [L-] Converting a string to bytes using bytes(string),Running out of gas can occur if the string is excessively large

If the gas required for the string-to-bytes conversion exceeds the gas limit set for a particular transaction or block, the transaction will fail due to an out-of-gas exception. The exact gas limit can vary based on the network and configuration

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

440: require(bytes(name).length <= 100, "Pool's name is too large");

```

### Recommended Mitigation

String lengths to ensure it stays within acceptable gas limits and to find the optimal chunk size for your specific use case


## [L-2] LACK OF CHECKS THE INTEGER RANGES

Consider edge cases, and conduct security audits to identify potential vulnerabilities or issues related to integer ranges. Taking a proactive approach to ensure the correctness and safety of your code is essential for building secure smart contracts

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Comptroller.sol

loopLimit is not checked before set _setMaxLoopsLimit. loopLimit can be zero value. Should be checked before set _setMaxLoopsLimit

function initialize(uint256 loopLimit, address accessControlManager) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager);

        _setMaxLoopsLimit(loopLimit);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L143

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1350-L1396

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Rewards/RewardsDistributor.sol

loopLimit_ is not checked before set _setMaxLoopsLimit. loopLimit_ can be zero value. Should be checked before set _setMaxLoopsLimit

function initialize(
        Comptroller comptroller_,
        IERC20Upgradeable rewardToken_,
        uint256 loopsLimit_,
        address accessControlManager_
    ) external initializer {
        comptroller = comptroller_;
        rewardToken = rewardToken_;
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager_);

        _setMaxLoopsLimit(loopsLimit_);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L111-L123

### Recommended Mitigation

```solidity

Require(loopLimit > 0, "the loopLimit  can't be zero value ");

```

##

## [L-] Missing Event for initialize

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1350-L1396

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L131-L150

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L111-L123

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L164-L180

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L73-L93

### Recommendation: 

Add Event-Emit

##

## [L-3] Unsafe typecasting method is used to cast bytes32 to uint256 

To convert a bytes32 variable to a uint256 variable in Solidity, you can use the uint256 cast with the abi.decode function

### EXAMPLE :

```solidity

```solidity

```
### Recommended Mitigation

Use safe casting way instead incompatible way 

##

## [L-8] Missing events for critical parameter changes

Events help non-contract tools to track changes, and events prevent users from being surprised by changes


##

## [L-] Upgrade OpenZeppelin Contract/contracts-upgradeable Dependency

An outdated OZ version 4.8.0 is used (which has known vulnerabilities, see: https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.8.0)

```
FILE: package.json

51: "@openzeppelin/contracts": "^4.8.0",
52: "@openzeppelin/contracts-upgradeable": "^4.8.0",


``` 
### Recommended Mitigation Steps
Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.3 via >=4.8.3

##

## [L-] Minting tokens to the zero address should be avoided

The core function mint is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. Address(0) check is missing in both this function and the internal function _mint, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address


##

## [L-14] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN 

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-] Even with the onlyOwner modifier, it is best practice to use the re-entrancy pattern

It's still good practice to apply the reentry model as a precautionary measure in case the code is changed in the future to remove the onlyOwner modifier or the contract is used as a base contract for other contracts.

Using the reentry modifier provides an additional layer of security and ensures that your code is protected from potential reentry attacks regardless of any other security measures you take.

So even if you followed the "check-effects-interactions" pattern correctly, it's still considered good practice to use the reentry modifier

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Comptroller.sol

927: function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner {
961: function setPriceOracle(PriceOracle newOracle) external onlyOwner {
973: function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL927C1-L927C96

```solidity
FILE: 2023-05-venus/contracts/VToken.sol

505: function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {
515: function setShortfallContract(address shortfall_) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL505C4-L505C97

```solidity
FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

348: function updatePoolRegistry(address _poolRegistry) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL348C5-L348C76

```solidity
FILE: 2023-05-venus/contracts/Rewards/RewardsDistributor.sol

181: function grantRewardToken(address recipient, uint256 amount) external onlyOwner {
219: function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {
249: function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL181C4-L181C86

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

188:  function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {
198:  function setShortfallContract(Shortfall shortfall_) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL188C4-L188C97

```solidity
FILE: 2023-05-venus/contracts/RiskFund/RiskFund.sol

99:  function setPoolRegistry(address _poolRegistry) external onlyOwner {
110: function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {
126: function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {
205: function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL99C1-L100C1


### Recommended Mitigation:

```solidity
  modifier noReentrant() {
        require(!locked, "Reentrant call");
        locked = true;
        _;
        locked = false;
    }
```

##

## [L-] Front running attacks by the onlyOwner



##

## [L-] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead. If all arguments are strings and or bytes, bytes.concat() should be used instead

##

## [L-1] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256

Using the SafeCast library can help prevent unexpected errors in your Solidity code and make your contracts more secure


### Recommended Mitigation Steps:
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

##



##

## [L-] Events are missing sender information

When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender the events of these types of action will make events much more useful to end users.

##

## [L-] Use Ownable2Step's transfer function rather than Ownable's for transfers of ownership

[Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/access/Ownable2Step.sol#L31-L56) and [Ownable2StepUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/Ownable2StepUpgradeable.sol#L47-L63) prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own


##

## [L-] Upgradeable contract not initialized

Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

File: src/contracts/core/StrategyManager.sol

/// @audit missing __Ownable_init()
26:   contract StrategyManager is

/// @audit missing __ReentrancyGuard_init()
26:   contract StrategyManager is
https://github.com/code-423n4/2023-04-eigenlayer/blob/398cc428541b91948f717482ec973583c9e76232/src/contracts/core/StrategyManager.sol#L26

File: src/contracts/pods/DelayedWithdrawalRouter.sol

/// @audit missing __Ownable_init()
11:   contract DelayedWithdrawalRouter is Initializable, OwnableUpgradeable, ReentrancyGuardUpgradeable, Pausable, IDelayedWithdrawalRouter {

/// @audit missing __ReentrancyGuard_init()
11:   contract DelayedWithdrawalRouter is Initializable, OwnableUpgradeable, ReentrancyGuardUpgradeable, Pausable, IDelayedWith

##

## [L-]  Unsafe ERC20 operation(s)

It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert

##

## [L-] Unbounded loop

New items are pushed into the challenges array. Currently, the array can grow indefinitely. E.g. there's no maximum limit and there's no functionality to remove array values.

If the array grows too large, calling functions that use challenges might run out of gas and revert.

##

## [L-] Use safeTransferOwnership instead of transferOwnership function

Use safeTransferOwnership which is safer. Use it as it is more secure due to 2-stage ownership transfer

##

## [L-]  INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY (Initializer could be front-run)

initialize() function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the __Ownable2Step_init() function.

### initialize() function

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

138: function initialize(uint256 loopLimit, address accessControlManager) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138

```solidity
FILE: 2023-05-venus/contracts/VToken.sol

58: function initialize(
        address underlying_,
        ComptrollerInterface comptroller_,
        InterestRateModel interestRateModel_,
        uint256 initialExchangeRateMantissa_,
        string memory name_,
        string memory symbol_,
        uint8 decimals_,
        address admin_,
        address accessControlManager_,
        RiskManagementInit memory riskManagement,
        uint256 reserveFactorMantissa_
    ) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL59C4-L71C29

```solidity
FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

131: function initialize(
        address convertibleBaseAsset_,
        IRiskFund riskFund_,
        uint256 minimumPoolBadDebt_,
        address accessControlManager_
    ) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL131C4-L136C29

```solidity
FILE: 2023-05-venus/contracts/Rewards/RewardsDistributor.sol

111: function initialize(
        Comptroller comptroller_,
        IERC20Upgradeable rewardToken_,
        uint256 loopsLimit_,
        address accessControlManager_
    ) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL111C5-L116C29

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

164: function initialize(
        VTokenProxyFactory vTokenFactory_,
        JumpRateModelFactory jumpRateFactory_,
        WhitePaperInterestRateModelFactory whitePaperFactory_,
        Shortfall shortfall_,
        address payable protocolShareReserve_,
        address accessControlManager_
    ) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL164C5-L171C29

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/RiskFund/RiskFund.sol

73: function initialize(
        address pancakeSwapRouter_,
        uint256 minAmountToConvert_,
        address convertibleBaseAsset_,
        address accessControlManager_,
        uint256 loopsLimit_
    ) external initializer {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL73C5-L79C29


### Recommended Mitigation Steps

Add a control that makes initialize() only call the Deployer Contract;

```solidity

if (msg.sender != DEPLOYER_ADDRESS) {
				revert NotDeployer();
				}

```

##

## [L-] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues






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

##

## [NC-] Reduce the inheritance list

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L19-L25

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L19-L24

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L17-L23





























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