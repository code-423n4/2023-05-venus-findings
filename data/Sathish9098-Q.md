## LOW FINDINGS 

| Issue Count | Issues | Instances |
|-----------------|-----------------|-----------------|
| [L-1] | Converting a string to bytes using bytes(string),Running out of gas can occur if the string is excessively large   |  1 |
| [L-2] | LACK OF CHECKS THE INTEGER RANGES   | 6  |
| [L-3] | Missing Event for initialize   | 5  |
| [L-4] | Upgrade OpenZeppelin Contract/contracts-upgradeable Dependency    | 2  |
| [L-5] | Even with the onlyOwner modifier, it is best practice to use the re-entrancy pattern  | 15  |
| [L-6] | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY (Initializer could be front-run)  | 6  |
| [L-7] |  Use safeTransferOwnership instead of transferOwnership function | 2  |
| [L-8] | DECIMALS() NOT PART OF ERC20 STANDARD | 2  |
| [L-9] | Missing ReEntrancy Guard to external functions when transfer tokens  | 2  |
| [L-10] | address() function is only available in Solidity code  | 8  |
| [L-11] | Insufficient coverage  | -  |
| [L-12] | Project Upgrade and Stop Scenario should be  | -  |

# NON CRITICAL FINDINGS

| Issue Count | Issues | Instances |
|-----------------|-----------------|-----------------|
| [NC-1]  | Add a timelock to critical functions  |  15 |
| [NC-2]  | No SAME VALUE INPUT CONTROL  |  15 |
| [NC-3]  |  Critical changes should use two-step procedure | 15  |
| [NC-4]  | For functions, follow Solidity standard naming conventions (internal function style rule)  |  4 |
| [NC-5]  | Use a more recent version of solidity  |  - |
| [NC-6]  | Contract layout and order of functions  | -  |
| [NC-7]  | Tokens accidentally sent to the contract cannot be recovered  |  - |
| [NC-8]  | Use SMTChecker  |  - |
| [NC-9]  | According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword |  - |
| [NC-10]  | Reduce the inheritance list  | 5  |
| [NC-11]  | Need Fuzz testing for unchecked  | 2  |

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

## [L-3] Missing Events for initialize

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

## [L-5] Even with the onlyOwner modifier, it is best practice to use the re-entrancy pattern

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

## [L-6] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY (Initializer could be front-run)

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

## [L-7] Use safeTransferOwnership instead of transferOwnership function

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

## [L-8] DECIMALS() NOT PART OF ERC20 STANDARD

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

284: uint256 underlyingDecimals = IERC20Metadata(input.asset).decimals();

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL284C7-L284C77

```solidity
FILE: 2023-05-venus/contracts/Lens/PoolLens.sol

361: underlyingDecimals = IERC20Metadata(vToken.underlying()).decimals();

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL361C9-L361C77


##

## [L-9] Missing ReEntrancy Guard to external functions when transfer tokens using external calls 

If there are external calls, particularly involving token transfers or interactions with other contracts, there may be a potential for reentrancy vulnerabilities. Even with check-effect-interaction pattern the ReEntrancyGuard gives extra layer of security.

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L190-L199

claimRewardToken function transfer the tokens using  _grantRewardToken internal functions.

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Rewards/RewardsDistributor.sol

277: function claimRewardToken(address holder, VToken[] memory vTokens) public {
        uint256 vTokensCount = vTokens.length;

        _ensureMaxLoops(vTokensCount);

        for (uint256 i; i < vTokensCount; ++i) {
            VToken vToken = vTokens[i];
            require(comptroller.isMarketListed(vToken), "market must be listed");
            Exp memory borrowIndex = Exp({ mantissa: vToken.borrowIndex() });
            _updateRewardTokenBorrowIndex(address(vToken), borrowIndex);
            _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
            _updateRewardTokenSupplyIndex(address(vToken));
            _distributeSupplierRewardToken(address(vToken), holder);
        }
        rewardTokenAccrued[holder] = _grantRewardToken(holder, rewardTokenAccrued[holder]); //@audit Token transfer call 
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L277-L292

### Recommended Mitigation Steps
Use Openzeppelin Re-Entrancy pattern.

##

## [L-10] address() function is only available in Solidity code

Address() function cannot be used outside the context of a contract. If you are interacting with the contract externally, such as through a web3 library or a command-line interface this address() function method is not compatible.

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

286: _updateRewardTokenBorrowIndex(address(vToken), borrowIndex);
287: _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
288: _updateRewardTokenSupplyIndex(address(vToken));
289: _distributeSupplierRewardToken(address(vToken), holder);
315: _updateRewardTokenSupplyIndex(address(vToken));
311:  if (rewardTokenSupplySpeeds[address(vToken)] != supplySpeed) {
318: rewardTokenSupplySpeeds[address(vToken)] = supplySpeed;
330: rewardTokenBorrowSpeeds[address(vToken)] = borrowSpeed;


```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L286-L289

Recommended Mitigation:

Need to use the appropriate methods or functions provided by those tools to obtain the contract address

##

## [L-11] Insufficient coverage

Due to its capacity, test coverage is expected to be 100%.

```
 What is the overall line coverage percentage provided by your tests?: 94.21
```

##

## [L-12] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol





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

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L13-L20

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

## [NC-7] Tokens accidentally sent to the contract cannot be recovered

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
































