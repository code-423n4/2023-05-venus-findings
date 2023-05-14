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

## [L-1] Converting a string to bytes using bytes(string),Running out of gas can occur if the string is excessively large

If the gas required for the string-to-bytes conversion exceeds the gas limit set for a particular transaction or block, the transaction will fail due to an out-of-gas exception. The exact gas limit can vary based on the network and configuration

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

440: require(bytes(name).length <= 100, "Pool's name is too large");

```

### Recommended Mitigation

String lengths to ensure it stays within acceptable gas limits and to find the optimal chunk size for your specific use case

##

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

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L65-L77

### Recommended Mitigation

```solidity

Require(loopLimit > 0, "the loopLimit  can't be zero value ");

```

##

## [L-3] Missing Event for initialize

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1350-L1396

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L131-L150

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L111-L123

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L164-L180

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L73-L93

### Recommendation: 

Add Event-Emit


##

## [L-4] Upgrade OpenZeppelin Contract/contracts-upgradeable Dependency

An outdated OZ version 4.8.0 is used (which has known vulnerabilities, see: https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.8.0)

```
FILE: package.json

51: "@openzeppelin/contracts": "^4.8.0",
52: "@openzeppelin/contracts-upgradeable": "^4.8.0",


``` 
### Recommended Mitigation Steps
Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.3 via >=4.8.3

##

## [L-5] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN 

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


##

## [L-6] Even with the onlyOwner modifier, it is best practice to use the re-entrancy pattern

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

## [L-7]  INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY (Initializer could be front-run)

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

## [L-8] Use safeTransferOwnership instead of transferOwnership function

Use safeTransferOwnership which is safer. Use it as it is more secure due to 2-stage ownership transfer

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/VToken.sol

1395: _transferOwnership(admin_);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1395C9-L1395C36

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

245: comptrollerProxy.transferOwnership(msg.sender);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L245

##

## [L-] Minting tokens to the zero address should be avoided

The core function mint is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. Address(0) check is missing in both this function and the internal function _mint, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address

















## NON CRITICAL FINDINGS


##

## [NC-1] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to


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



##

## [NC-2] No same value control 

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

function setPriceOracle(PriceOracle newOracle) external onlyOwner {
        require(address(newOracle) != address(0), "invalid price oracle address");

        PriceOracle oldOracle = oracle;
        oracle = newOracle;
        emit NewPriceOracle(oldOracle, newOracle);
    }

function setMaxLoopsLimit(uint256 limit) external onlyOwner {
        _setMaxLoopsLimit(limit);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L961-L967


```solidity
FILE: FILE: 2023-05-venus/contracts/VToken.sol

1407: function _setProtocolShareReserve(address payable protocolShareReserve_) internal {
        if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
        address oldProtocolShareReserve = address(protocolShareReserve);
        protocolShareReserve = protocolShareReserve_;
        emit NewProtocolShareReserve(oldProtocolShareReserve, address(protocolShareReserve_));
    }

1398: function _setShortfallContract(address shortfall_) internal {
        if (shortfall_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
        address oldShortfall = shortfall;
        shortfall = shortfall_;
        emit NewShortfallContract(oldShortfall, shortfall_);
    }


```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1407-L1414

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


##

## [NC-3] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address


```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

function setPriceOracle(PriceOracle newOracle) external onlyOwner {
        require(address(newOracle) != address(0), "invalid price oracle address");

        PriceOracle oldOracle = oracle;
        oracle = newOracle;
        emit NewPriceOracle(oldOracle, newOracle);
    }

function setMaxLoopsLimit(uint256 limit) external onlyOwner {
        _setMaxLoopsLimit(limit);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L961-L967


```solidity
FILE: FILE: 2023-05-venus/contracts/VToken.sol

1407: function _setProtocolShareReserve(address payable protocolShareReserve_) internal {
        if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
        address oldProtocolShareReserve = address(protocolShareReserve);
        protocolShareReserve = protocolShareReserve_;
        emit NewProtocolShareReserve(oldProtocolShareReserve, address(protocolShareReserve_));
    }

1398: function _setShortfallContract(address shortfall_) internal {
        if (shortfall_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
        address oldShortfall = shortfall;
        shortfall = shortfall_;
        emit NewShortfallContract(oldShortfall, shortfall_);
    }


```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1407-L1414

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


##

## [NC-4] For functions, follow Solidity standard naming conventions (internal function style rule)

Description
The bellow codes don’t follow Solidity’s standard naming convention,

internal and private functions and variables : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L453-L458

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L475-L479

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L495-L501

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L516-L521

##

## [NC-5] Use a more recent version of solidity

Latest solidity version is 0.8.19 

CONTEXT
ALL CONTRACT SCOPES


##

## [NC-6] Contract layout and order of functions

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

## [Nc-7] Tokens accidentally sent to the contract cannot be recovered

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

## [NC-8] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##

## [NC-9] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L72

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L23-L50

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L98-L108

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L74-L80

##

## [NC-10] Reduce the inheritance list

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L17-L24

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L19-L24

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L17

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L12

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L19-L25

##

## [NC-11] Need Fuzz testing for unchecked 

```solidity
FILE: 2023-05-venus/contracts/RiskFund/ReserveHelpers.sol

63: unchecked {
                balanceDifference = currentBalance - assetReserve;
            }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#LL63C13-L65C14

```solidity
FILE: 2023-05-venus/contracts/BaseJumpRateModelV2.sol

184: unchecked {
                excessUtil = util - kink;
            }


```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L184-L186

































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