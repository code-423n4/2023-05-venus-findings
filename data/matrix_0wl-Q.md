## Non Critical Issues

|       | Issue                                                               |
| ----- | :------------------------------------------------------------------ |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                |
| NC-2  | GENERATE PERFECT CODE HEADERS EVERY TIME                            |
| NC-3  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                     |
| NC-4  | FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE                   |
| NC-5  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES             |
| NC-6  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS        |
| NC-7  | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION               |
| NC-8  | NO SAME VALUE INPUT CONTROL                                         |
| NC-9  | Omissions in events                                                 |
| NC-10 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                  |
| NC-11 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE |
| NC-12 | USE A MORE RECENT VERSION OF SOLIDITY                               |
| NC-13 | USE UNDERSCORES FOR NUMBER LITERALS                                 |
| NC-14 | Event is missing `indexed` fields                                   |
| NC-15 | Functions not used internally could be marked external              |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

726:     function setCollateralFactor(

779:     function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {

836:     function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {

862:     function setMarketSupplyCaps(VToken[] calldata vTokens, uint256[] calldata newSupplyCaps) external {

885:     function setActionsPaused(

912:     function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {

961:     function setPriceOracle(PriceOracle newOracle) external onlyOwner {

973:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

1224:     function _setActionPaused(

1381:     function _getCollateralFactor(VToken asset) internal view returns (Exp memory) {

1390:     function _getLiquidationThreshold(VToken asset) internal view returns (Exp memory) {

```

```solidity
File: MaxLoopsLimitHelper.sol

25:     function _setMaxLoopsLimit(uint256 limit) internal {

```

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

198:     function setShortfallContract(Shortfall shortfall_) external onlyOwner {

332:     function setPoolName(address comptroller, string calldata name) external {

```

```solidity
File: Rewards/RewardsDistributor.sol

197:     function setRewardTokenSpeeds(

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {

249:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

304:     function _setRewardTokenSpeed(

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

```

```solidity
File: RiskFund/RiskFund.sol

99:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

110:     function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {

126:     function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {

137:     function setMinAmountToConvert(uint256 minAmountToConvert_) external {

205:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```

```solidity
File: VToken.sol

310:     function setProtocolSeizeShare(uint256 newProtocolSeizeShareMantissa_) external {

330:     function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {

369:     function setInterestRateModel(InterestRateModel newInterestRateModel) external override {

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

515:     function setShortfallContract(address shortfall_) external onlyOwner {

1138:     function _setComptroller(ComptrollerInterface newComptroller) internal {

1154:     function _setReserveFactorFresh(uint256 newReserveFactorMantissa) internal {

1243:     function _setInterestRateModelFresh(InterestRateModel newInterestRateModel) internal {

1398:     function _setShortfallContract(address shortfall_) internal {

1407:     function _setProtocolShareReserve(address payable protocolShareReserve_) internal {

```

```solidity
File: VTokenInterfaces.sol

325:     function setReserveFactor(uint256 newReserveFactorMantissa) external virtual;

333:     function setInterestRateModel(InterestRateModel newInterestRateModel) external virtual;

```

### [NC-2] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-3] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

23:     uint256 public constant blocksPerYear = 10512000;

```

```solidity
File: ErrorReporter.sol

5:     uint256 public constant NO_ERROR = 0; // support legacy return codes

```

```solidity
File: InterestRateModel.sol

10:     bool public constant isInterestRateModel = true;

```

```solidity
File: Rewards/RewardsDistributor.sol

29:     uint224 public constant rewardTokenInitialIndex = 1e36;

```

```solidity
File: VTokenInterfaces.sol

142:     bool public constant isVToken = true;

```

```solidity
File: WhitePaperInterestRateModel.sol

17:     uint256 public constant blocksPerYear = 2102400;

```

### [NC-4] FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

#### Description:

Use camel case for all functions, parameters and variables.

#### **Proof Of Concept**

```solidity
File: ExponentialNoError.sol

38:     function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {

47:     function mul_ScalarTruncateAddUInt(

```

### [NC-5] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

4: import "@venusprotocol/governance-contracts/contracts/Governance/IAccessControlManagerV8.sol";

5: import "./InterestRateModel.sol";

```

```solidity
File: Comptroller.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

6: import "./VToken.sol";

7: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

8: import "./ComptrollerInterface.sol";

9: import "./ComptrollerStorage.sol";

10: import "./Rewards/RewardsDistributor.sol";

11: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";

12: import "./MaxLoopsLimitHelper.sol";

```

```solidity
File: ComptrollerInterface.sol

4: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

5: import "./VToken.sol";

6: import "./Rewards/RewardsDistributor.sol";

```

```solidity
File: ComptrollerStorage.sol

4: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

5: import "./VToken.sol";

```

```solidity
File: Factories/JumpRateModelFactory.sol

4: import "../JumpRateModelV2.sol";

```

```solidity
File: Factories/VTokenProxyFactory.sol

4: import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";

6: import "../VToken.sol";

7: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";

8: import "../VTokenInterfaces.sol";

```

```solidity
File: Factories/WhitePaperInterestRateModelFactory.sol

4: import "../WhitePaperInterestRateModel.sol";

```

```solidity
File: JumpRateModelV2.sol

4: import "./BaseJumpRateModelV2.sol";

```

```solidity
File: Lens/PoolLens.sol

4: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

5: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

7: import "../VToken.sol";

8: import "../ComptrollerInterface.sol";

9: import "../Pool/PoolRegistryInterface.sol";

10: import "../Pool/PoolRegistry.sol";

```

```solidity
File: Pool/PoolRegistry.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

6: import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";

7: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

9: import "../Comptroller.sol";

10: import "../Factories/VTokenProxyFactory.sol";

11: import "../Factories/JumpRateModelFactory.sol";

12: import "../Factories/WhitePaperInterestRateModelFactory.sol";

13: import "../WhitePaperInterestRateModel.sol";

14: import "../VToken.sol";

15: import "../InterestRateModel.sol";

16: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";

17: import "../Shortfall/Shortfall.sol";

18: import "../VTokenInterfaces.sol";

19: import "./PoolRegistryInterface.sol";

```

```solidity
File: Proxy/UpgradeableBeacon.sol

4: import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";

```

```solidity
File: Rewards/RewardsDistributor.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

6: import "../ExponentialNoError.sol";

7: import "../VToken.sol";

8: import "../Comptroller.sol";

9: import "../MaxLoopsLimitHelper.sol";

10: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import "../ExponentialNoError.sol";

8: import "./IRiskFund.sol";

9: import "./ReserveHelpers.sol";

10: import "./IProtocolShareReserve.sol";

```

```solidity
File: RiskFund/ReserveHelpers.sol

4: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

6: import "../ComptrollerInterface.sol";

7: import "../Pool/PoolRegistryInterface.sol";

```

```solidity
File: RiskFund/RiskFund.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import "../VToken.sol";

8: import "../Pool/PoolRegistry.sol";

9: import "../IPancakeswapV2Router.sol";

10: import "./ReserveHelpers.sol";

11: import "./IRiskFund.sol";

12: import "../Shortfall/IShortfall.sol";

13: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";

14: import "../MaxLoopsLimitHelper.sol";

```

```solidity
File: Shortfall/Shortfall.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

6: import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";

7: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

9: import "../VToken.sol";

10: import "../ComptrollerInterface.sol";

11: import "../RiskFund/IRiskFund.sol";

12: import "./IShortfall.sol";

13: import "../Pool/PoolRegistry.sol";

14: import "../Pool/PoolRegistryInterface.sol";

15: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";

```

```solidity
File: VToken.sol

4: import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";

5: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

7: import "./ComptrollerInterface.sol";

8: import "./VTokenInterfaces.sol";

9: import "./ErrorReporter.sol";

10: import "./InterestRateModel.sol";

11: import "./ExponentialNoError.sol";

12: import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";

13: import "./RiskFund/IProtocolShareReserve.sol";

```

```solidity
File: VTokenInterfaces.sol

4: import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

5: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

6: import "./ComptrollerInterface.sol";

7: import "./InterestRateModel.sol";

8: import "./ErrorReporter.sol";

```

```solidity
File: WhitePaperInterestRateModel.sol

4: import "./InterestRateModel.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-6] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

```

```solidity
File: Pool/PoolRegistry.sol

164:     function initialize(

```

```solidity
File: Rewards/RewardsDistributor.sol

111:     function initialize(

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

```

```solidity
File: RiskFund/RiskFund.sol

73:     function initialize(

```

```solidity
File: Shortfall/Shortfall.sol

131:     function initialize(

```

```solidity
File: VToken.sol

59:     function initialize(

```

### [NC-7] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

OUse (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

#### **Proof Of Concept**

```solidity
File: Shortfall/Shortfall.sol

60:     uint256 private constant MAX_BPS = 10000;

```

### [NC-8] NO SAME VALUE INPUT CONTROL

#### Description:

INPUT like address oracle should be inputAdd code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

#### **Proof Of Concept**

```solidity
File: RiskFund/ProtocolShareReserve.sol

45:         protocolIncome = _protocolIncome;

46:         riskFund = _riskFund;

56:         poolRegistry = _poolRegistry;

```

```solidity
File: RiskFund/RiskFund.sol

102:         poolRegistry = _poolRegistry;

```

```solidity
File: Shortfall/Shortfall.sol

297:         nextBidderBlockLimit = _nextBidderBlockLimit;

311:         incentiveBps = _incentiveBps;

324:         minimumPoolBadDebt = _minimumPoolBadDebt;

337:         waitForFirstBidder = _waitForFirstBidder;

351:         poolRegistry = _poolRegistry;

```

### [NC-9] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:.

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

823:         emit MarketSupported(vToken);

```

```solidity
File: Factories/VTokenProxyFactory.sol

47:         emit VTokenProxyDeployed(input);

```

```solidity
File: Rewards/RewardsDistributor.sol

149:         emit MarketInitialized(vToken);

450:         emit RewardTokenSupplyIndexUpdated(vToken);

```

```solidity
File: VToken.sol

530:         emit SweepToken(address(token));

```

### [NC-10] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

  solidity: {
    compilers: [
      {
        version: "0.8.13",
        settings: {
          optimizer: {
            enabled: true,
            details: {
              yul: !process.env.CI,
            },
          },
          outputSelection: {
            "*": {
              "*": ["storageLayout"],
            },
          },
        },
      },
      {
        version: "0.6.6",
        settings: {
          optimizer: {
            enabled: true,
            details: {
              yul: !process.env.CI,
            },
          },
          outputSelection: {
            "*": {
              "*": ["storageLayout"],
            },
          },
        },
      },
    ],
  },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-11] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

#### Context:

All Contracts

#### Description:

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last

### [NC-12] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Comptroller.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ComptrollerInterface.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ComptrollerStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ErrorReporter.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ExponentialNoError.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/JumpRateModelFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/VTokenProxyFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/WhitePaperInterestRateModelFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: IPancakeswapV2Router.sol

2: pragma solidity 0.8.13;

```

```solidity
File: InterestRateModel.sol

2: pragma solidity 0.8.13;

```

```solidity
File: JumpRateModelV2.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Lens/PoolLens.sol

2: pragma solidity 0.8.13;

```

```solidity
File: MaxLoopsLimitHelper.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Pool/PoolRegistry.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Pool/PoolRegistryInterface.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Proxy/UpgradeableBeacon.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Rewards/RewardsDistributor.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/IProtocolShareReserve.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/IRiskFund.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/ReserveHelpers.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/RiskFund.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Shortfall/IShortfall.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Shortfall/Shortfall.sol

2: pragma solidity 0.8.13;

```

```solidity
File: VToken.sol

2: pragma solidity 0.8.13;

```

```solidity
File: VTokenInterfaces.sol

2: pragma solidity 0.8.13;

```

```solidity
File: WhitePaperInterestRateModel.sol

2: pragma solidity 0.8.13;

```

### [NC-13] USE UNDERSCORES FOR NUMBER LITERALS

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

23:     uint256 public constant blocksPerYear = 10512000;

72:         require(address(accessControlManager_) != address(0), "invalid ACM address");

```

```solidity
File: Shortfall/Shortfall.sol

60:     uint256 private constant MAX_BPS = 10000;

149:         incentiveBps = 1000;

```

```solidity
File: WhitePaperInterestRateModel.sol

17:     uint256 public constant blocksPerYear = 2102400;

```

### [NC-14] Event is missing `indexed` fields

#### Description:

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.Parameters of certain events are expected to be indexed (e.g. ERC20 Transfer/Approval events) so that they’re included in the block’s bloom filter for faster access. Failure to do so might confuse off-chain tooling looking for such indexed events.

https://github.com/crytic/slither/wiki/Detector-Documentation#unindexed-erc20-event-oarameters

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

45:     event NewInterestParams(

```

```solidity
File: Comptroller.sol

30:     event MarketEntered(VToken vToken, address account);

33:     event MarketExited(VToken vToken, address account);

36:     event NewCloseFactor(uint256 oldCloseFactorMantissa, uint256 newCloseFactorMantissa);

39:     event NewCollateralFactor(VToken vToken, uint256 oldCollateralFactorMantissa, uint256 newCollateralFactorMantissa);

42:     event NewLiquidationThreshold(

49:     event NewLiquidationIncentive(uint256 oldLiquidationIncentiveMantissa, uint256 newLiquidationIncentiveMantissa);

52:     event NewPriceOracle(PriceOracle oldPriceOracle, PriceOracle newPriceOracle);

55:     event ActionPausedMarket(VToken vToken, Action action, bool pauseState);

58:     event NewBorrowCap(VToken indexed vToken, uint256 newBorrowCap);

61:     event NewMinLiquidatableCollateral(uint256 oldMinLiquidatableCollateral, uint256 newMinLiquidatableCollateral);

64:     event NewSupplyCap(VToken indexed vToken, uint256 newSupplyCap);

70:     event MarketSupported(VToken vToken);

```

```solidity
File: Factories/VTokenProxyFactory.sol

26:     event VTokenProxyDeployed(VTokenArgs args);

```

```solidity
File: MaxLoopsLimitHelper.sol

16:     event MaxLoopsLimitUpdated(uint256 oldMaxLoopsLimit, uint256 newmaxLoopsLimit);

```

```solidity
File: Pool/PoolRegistry.sol

113:     event PoolRegistered(address indexed comptroller, VenusPool pool);

118:     event PoolNameSet(address indexed comptroller, string oldName, string newName);

123:     event PoolMetadataUpdated(

132:     event MarketAdded(address indexed comptroller, address vTokenAddress);

```

```solidity
File: Rewards/RewardsDistributor.sol

57:     event DistributedSupplierRewardToken(

66:     event DistributedBorrowerRewardToken(

75:     event RewardTokenSupplySpeedUpdated(VToken indexed vToken, uint256 newSpeed);

78:     event RewardTokenBorrowSpeedUpdated(VToken indexed vToken, uint256 newSpeed);

81:     event RewardTokenGranted(address recipient, uint256 amount);

84:     event ContributorRewardTokenSpeedUpdated(address indexed contributor, uint256 newSpeed);

87:     event MarketInitialized(address vToken);

90:     event RewardTokenSupplyIndexUpdated(address vToken);

93:     event RewardTokenBorrowIndexUpdated(address vToken, Exp marketBorrowIndex);

96:     event ContributorRewardsUpdated(address contributor, uint256 rewardAccrued);

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

22:     event FundsReleased(address comptroller, address asset, uint256 amount);

```

```solidity
File: RiskFund/ReserveHelpers.sol

30:     event AssetsReservesUpdated(address indexed comptroller, address indexed asset, uint256 amount);

```

```solidity
File: RiskFund/RiskFund.sol

47:     event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);

50:     event MinAmountToConvertUpdated(uint256 oldMinAmountToConvert, uint256 newMinAmountToConvert);

53:     event SwappedPoolsAssets(address[] markets, uint256[] amountsOutMin, uint256 totalAmount);

56:     event TransferredReserveForAuction(address comptroller, uint256 amount);

```

```solidity
File: Shortfall/Shortfall.sol

75:     event AuctionStarted(

86:     event BidPlaced(address indexed comptroller, uint256 auctionStartBlock, uint256 bidBps, address indexed bidder);

89:     event AuctionClosed(

100:     event AuctionRestarted(address indexed comptroller, uint256 auctionStartBlock);

106:     event MinimumPoolBadDebtUpdated(uint256 oldMinimumPoolBadDebt, uint256 newMinimumPoolBadDebt);

109:     event WaitForFirstBidderUpdated(uint256 oldWaitForFirstBidder, uint256 newWaitForFirstBidder);

112:     event NextBidderBlockLimitUpdated(uint256 oldNextBidderBlockLimit, uint256 newNextBidderBlockLimit);

115:     event IncentiveBpsUpdated(uint256 oldIncentiveBps, uint256 newIncentiveBps);

```

```solidity
File: VTokenInterfaces.sol

149:     event AccrueInterest(uint256 cashPrior, uint256 interestAccumulated, uint256 borrowIndex, uint256 totalBorrows);

154:     event Mint(address indexed minter, uint256 mintAmount, uint256 mintTokens, uint256 accountBalance);

159:     event Redeem(address indexed redeemer, uint256 redeemAmount, uint256 redeemTokens, uint256 accountBalance);

164:     event Borrow(address indexed borrower, uint256 borrowAmount, uint256 accountBorrows, uint256 totalBorrows);

169:     event RepayBorrow(

184:     event BadDebtIncreased(address indexed borrower, uint256 badDebtDelta, uint256 badDebtOld, uint256 badDebtNew);

191:     event BadDebtRecovered(uint256 badDebtOld, uint256 badDebtNew);

232:     event NewProtocolSeizeShare(uint256 oldProtocolSeizeShareMantissa, uint256 newProtocolSeizeShareMantissa);

237:     event NewReserveFactor(uint256 oldReserveFactorMantissa, uint256 newReserveFactorMantissa);

242:     event ReservesAdded(address indexed benefactor, uint256 addAmount, uint256 newTotalReserves);

247:     event ReservesReduced(address indexed admin, uint256 reduceAmount, uint256 newTotalReserves);

252:     event Transfer(address indexed from, address indexed to, uint256 amount);

257:     event Approval(address indexed owner, address indexed spender, uint256 amount);

262:     event HealBorrow(address payer, address borrower, uint256 repayAmount);

267:     event SweepToken(address token);

```

```solidity
File: WhitePaperInterestRateModel.sol

29:     event NewInterestParams(uint256 baseRatePerBlock, uint256 multiplierPerBlock);

```

### [NC-15] Functions not used internally could be marked external

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

1144:     function getRewardDistributors() public view returns (RewardsDistributor[] memory) {

```

```solidity
File: RiskFund/RiskFund.sol

214:     function updateAssetsState(address comptroller, address asset) public override(IRiskFund, ReserveHelpers) {

```

```solidity
File: VToken.sol

625:     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {

```

```solidity
File: WhitePaperInterestRateModel.sol

67:     function getSupplyRate(

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | Avoid `transfer()`/`send()` as reentrancy mitigations                          |
| L-2  | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                                 |
| L-3  | DO NOT USE DEPRECATED LIBRARY FUNCTIONS                                        |
| L-4  | DON’T USE OWNER AND TIMELOCK                                                   |
| L-5  | Initializers could be front-run                                                |
| L-6  | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                                 |
| L-7  | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS     |
| L-8  | UNIFY RETURN CRITERIA                                                          |
| L-9  | POTENTIAL LOSS OF FUNDS ON TOKENS WITH BIG SUPPLIES                            |
| L-10 | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-11 | UNSAFE CAST                                                                    |
| L-12 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                       |
| L-13 | USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION            |

### [L-1] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: VToken.sol

99:     function transfer(address dst, uint256 amount) external override nonReentrant returns (bool) {

```

```solidity
File: VTokenInterfaces.sol

311:     function transfer(address dst, uint256 amount) external virtual returns (bool);

```

### [L-2] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

430:     function _setProtocolShareReserve(address payable protocolShareReserve_) internal {

```

```solidity
File: Rewards/RewardsDistributor.sol

197:     function setRewardTokenSpeeds(

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

```

```solidity
File: RiskFund/RiskFund.sol

99:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

126:     function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {

```

```solidity
File: VToken.sol

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

515:     function setShortfallContract(address shortfall_) external onlyOwner {

1398:     function _setShortfallContract(address shortfall_) internal {

1407:     function _setProtocolShareReserve(address payable protocolShareReserve_) internal {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-3] DO NOT USE DEPRECATED LIBRARY FUNCTIONS

#### Description:

You use `safeApprove `of openZeppelin although it’s deprecated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38%3E).

You should change it to increase/decrease Allowance as OpenZeppilin says.

#### **Proof Of Concept**

```solidity
File: Pool/PoolRegistry.sol

322:         token.safeApprove(address(vToken), 0);

323:         token.safeApprove(address(vToken), amountToSupply);

```

```solidity
File: RiskFund/RiskFund.sol

258:                     IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);

259:                     IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);

```

### [L-4] DON’T USE OWNER AND TIMELOCK

#### Description:

Using a timelock contract gives confidence to the user, but when check onlyByOwnGov allow the owner and the timelock The owner manipulates the contract without a lock time period.

#### **Proof Of Concept**

```solidity
File: VToken.sol

525:         require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");

```

### [L-5] Initializers could be front-run

#### Description:

Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

139:         __Ownable2Step_init();

```

```solidity
File: Pool/PoolRegistry.sol

164:     function initialize(

171:     ) external initializer {

172:         __Ownable2Step_init();

```

```solidity
File: Rewards/RewardsDistributor.sol

111:     function initialize(

116:     ) external initializer {

119:         __Ownable2Step_init();

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

43:         __Ownable2Step_init();

```

```solidity
File: RiskFund/RiskFund.sol

73:     function initialize(

79:     ) external initializer {

85:         __Ownable2Step_init();

```

```solidity
File: Shortfall/Shortfall.sol

131:     function initialize(

136:     ) external initializer {

141:         __Ownable2Step_init();

143:         __ReentrancyGuard_init();

```

```solidity
File: VToken.sol

59:     function initialize(

71:     ) external initializer {

75:         _initialize(

1350:     function _initialize(

1363:         __Ownable2Step_init();

```

### [L-6] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

```

```solidity
File: Pool/PoolRegistry.sol

171:     ) external initializer {

```

```solidity
File: Rewards/RewardsDistributor.sol

116:     ) external initializer {

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

```

```solidity
File: RiskFund/RiskFund.sol

79:     ) external initializer {

```

```solidity
File: Shortfall/Shortfall.sol

136:     ) external initializer {

```

```solidity
File: VToken.sol

71:     ) external initializer {

```

### [L-7] MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

```

```solidity
File: Pool/PoolRegistry.sol

164:     function initialize(

```

```solidity
File: Rewards/RewardsDistributor.sol

111:     function initialize(

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

```

```solidity
File: RiskFund/RiskFund.sol

73:     function initialize(

```

```solidity
File: Shortfall/Shortfall.sol

131:     function initialize(

```

```solidity
File: VToken.sol

59:     function initialize(

75:         _initialize(

1350:     function _initialize(

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-8] UNIFY RETURN CRITERIA

#### Description:

In files sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

154:     function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {

187:     function exitMarket(address vTokenAddress) external override returns (uint256) {

988:         returns (

1017:         returns (

1038:     function getAllMarkets() external view override returns (VToken[] memory) {

1047:     function isMarketListed(VToken vToken) external view returns (bool) {

1058:     function getAssetsIn(address account) external view returns (VToken[] memory) {

1070:     function checkMembership(address account, VToken vToken) external view returns (bool) {

1088:     ) external view override returns (uint256 error, uint256 tokensToSeize) {

1119:     function getRewardsByMarket(address vToken) external view returns (RewardSpeeds[] memory rewardSpeeds) {

1136:     function isComptroller() external pure override returns (bool) {

1144:     function getRewardDistributors() public view returns (RewardsDistributor[] memory) {

1154:     function actionPaused(address market, Action action) public view returns (bool) {

1164:     function isDeprecated(VToken vToken) public view returns (bool) {

1276:     function _getCurrentLiquiditySnapshot(address account, function(VToken) internal view returns (Exp memory) weight)

1279:         returns (AccountLiquiditySnapshot memory snapshot)

1291:          liquidation threshold. Accepts the address of the VToken and returns the weight

1301:         function(VToken) internal view returns (Exp memory) weight

1302:     ) internal view returns (AccountLiquiditySnapshot memory snapshot) {

1368:     function _safeGetUnderlyingPrice(VToken asset) internal view returns (uint256) {

1381:     function _getCollateralFactor(VToken asset) internal view returns (Exp memory) {

1390:     function _getLiquidationThreshold(VToken asset) internal view returns (Exp memory) {

1405:         returns (

```

```solidity
File: IPancakeswapV2Router.sol

11:     ) external returns (uint256[] memory amounts);

```

### [L-9] POTENTIAL LOSS OF FUNDS ON TOKENS WITH BIG SUPPLIES

#### Description:

`swap()` and `mint()` both reverts if either `2^112` or `2^128` tokens are sent to the pair. This would result in the funds being stuck and nobody being able to mint or swap. Submitting as low because the cost of attack is extremely high, but it’s good to be aware of it.

#### **Proof Of Concept**

```solidity
File: VToken.sol

180:     function mint(uint256 mintAmount) external override nonReentrant returns (uint256) {

```

```solidity
File: VTokenInterfaces.sol

271:     function mint(uint256 mintAmount) external virtual returns (uint256);

```

### [L-10] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: Rewards/RewardsDistributor.sol

419:             rewardToken.safeTransfer(user, amount);

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

81:         IERC20Upgradeable(asset).safeTransfer(protocolIncome, protocolIncomeAmount);

82:         IERC20Upgradeable(asset).safeTransfer(riskFund, amount - protocolIncomeAmount);

```

```solidity
File: RiskFund/RiskFund.sol

194:         IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);

```

```solidity
File: Shortfall/Shortfall.sol

183:                     erc20.safeTransfer(auction.highestBidder, previousBidAmount);

190:                     erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);

229:                 erc20.safeTransfer(address(auction.markets[i]), bidAmount);

232:                 erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);

248:         IERC20Upgradeable(convertibleBaseAsset).safeTransfer(auction.highestBidder, riskFundBidAmount);

```

```solidity
File: VToken.sol

528:         token.safeTransfer(owner(), balance);

1286:         token.safeTransfer(to, amount);

```

### [L-11] UNSAFE CAST

#### Description:

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

```solidity
File: ExponentialNoError.sol

70:         return uint32(n);

```

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

### [L-12] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.

```solidity
File: VToken.sol

99:     function transfer(address dst, uint256 amount) external override nonReentrant returns (bool) {

```

```solidity
File: VTokenInterfaces.sol

311:     function transfer(address dst, uint256 amount) external virtual returns (bool);

```

### [L-13] USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION

#### Description:

transferOwnership function is used to change Ownership from Owned.sol.

Use a 2 structure transferOwnership which is safer.

safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: Pool/PoolRegistry.sol

245:         comptrollerProxy.transferOwnership(msg.sender);

```

```solidity
File: VToken.sol

1395:         _transferOwnership(admin_);

```

#### Recommended Mitigation Steps:

Use Ownable2Step.sol
