# QA Report for Venus Protocol Isolated Pools contest
## Overview
During the audit, 1 low and 8 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Add a timelock to critical functions | Low | 20
NC-1 | Update import usages | Non-Critical | 1
NC-2 | Scientific notation may be used | Non-Critical | 2
NC-3 | Unused event| Non-Critical | 1
NC-4 | If possible, place if/require-statements at the top of the function | Non-Critical | 1
NC-5 | Prevent zero transfers | Non-Critical | 4
NC-6 | Lack of event emission in initialize() function | Non-Critical | 7
NC-7 | No same value input control | Non-Critical | 14
NC-8 | Missing leading underscores | Non-Critical | 64

## Low Risk Findings(1)
### L-1. Add a timelock to critical functions
##### Description
Giving users time to react and adjust to critical changes in protocol provides more guarantees and increases the transparency of the protocol.
##### Instances
- [```function setCloseFactor(uint256 newCloseFactorMantissa) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L702) 
- [```function setCollateralFactor(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L726) 
- [```function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L836) 
- [```function setMarketSupplyCaps(VToken[] calldata vTokens, uint256[] calldata newSupplyCaps) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L862) 
- [```function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L912) 
- [```function setPriceOracle(PriceOracle newOracle) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961) 
- [```function setMaxLoopsLimit(uint256 limit) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L973) 
- [```function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L330) 
- [```function setInterestRateModel(InterestRateModel newInterestRateModel) external override {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L369) 
- [```function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L505) 
- [```function setShortfallContract(address shortfall_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L515) 
- [```function setMaxLoopsLimit(uint256 limit) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L249) 
- [```function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L188) 
- [```function setShortfallContract(Shortfall shortfall_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L198) 
- [```function setPoolRegistry(address _poolRegistry) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99) 
- [```function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L110) 
- [```function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126) 
- [```function setMinAmountToConvert(uint256 minAmountToConvert_) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L137) 
- [```function setMaxLoopsLimit(uint256 limit) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L205) 
- [```function setPoolRegistry(address _poolRegistry) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L53) 

##### Recommendation
Consider adding a timelock.
#
## Non-Critical Risk Findings(8)
### NC-1. Update import usages
##### Description
For modern and more readable code, consider updating import usages.
##### Instances
- all contracts

##### Recommendation
Change to:
```import {contract1 , contract2} from "filename.sol";```
#
### NC-2. Scientific notation may be used
##### Description
For readability and to avoid misprints, it is better to use scientific notation.
##### Instances
- [```uint256 private constant MAX_BPS = 10000;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L60) 
- [```incentiveBps = 1000;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L149) 

##### Recommendation
Replace ```10000``` with ```10e4```.
#
### NC-3. Unused event
##### Description
The event ```AmountOutMinUpdated``` is not emitted.
##### Instances
- [```event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L47) 

##### Recommendation
Emit event or delete it.
#
### NC-4. If possible, place if/require-statements at the top of the function
##### Description
Validation of input parameters should be at the beginning of the function.
##### Instances
- [```require(newComptroller.isComptroller(), "marker method returned false");```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1141) 

#
### NC-5. Prevent zero transfers
##### Description
Check that amount to transfer > 0.
##### Instances
- [```token.safeTransferFrom(from, address(this), amount);```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1275) 
- [```token.safeTransfer(to, amount);```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1286) 
- [```token.safeTransferFrom(from, address(this), amount);```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L416) 
- [```IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L194) 

#
### NC-6. Lack of event emission in initialize() function
##### Description
Lack of event emission complicates recording the init parameters for off-chain monitoring and reduces transparency.
##### Instances
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L138
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L59
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L131
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L111
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L39

##### Recommendation
Consider emitting an event in the initialize() function
#
### NC-7. No same value input control
##### Instances
- [```function setCloseFactor(uint256 newCloseFactorMantissa) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L702) 
- [```function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L779) 
- [```function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L912) 
- [```function setPriceOracle(PriceOracle newOracle) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961) 
- [```function _setComptroller(ComptrollerInterface newComptroller) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1138) 
- [```function _setShortfallContract(address shortfall_) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1398) 
- [```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1407) 
- [```function _setShortfallContract(Shortfall shortfall_) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L421) 
- [```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L430) 
- [```function setPoolRegistry(address _poolRegistry) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99) 
- [```function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126) 
- [```function setMinAmountToConvert(uint256 minAmountToConvert_) external {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L137) 
- [```function setPoolRegistry(address _poolRegistry) external onlyOwner {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L53) 
- [```function _setMaxLoopsLimit(uint256 limit) internal {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25) 

##### Recommendation
Check ```if (variableA == parameterA) revert SameValue();```
#
### NC-8. Missing leading underscores
##### Description
Internal and private state variables, constants, immutables and functions should have a leading underscore.
##### Instances
- [```function updateMarketBorrowIndex(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L453) 
- [```function updateMarketSupplyIndex(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L475) 
- [```function calculateBorrowerReward(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L495) 
- [```function calculateSupplierReward(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L516) 
- [```IRiskFund private riskFund;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L51) 
- [```uint256 private incentiveBps;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L57) 
- [```uint256 private constant MAX_BPS = 10000;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L60) 
- [```Comptroller private comptroller;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L52) 
- [```address private pancakeSwapRouter;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L29) 
- [```uint256 private minAmountToConvert;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L30) 
- [```address private convertibleBaseAsset;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L31) 
- [```address private shortfall;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L32) 
- [```uint256 internal constant borrowRateMaxMantissa = 0.0005e16;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L53) 
- [```uint256 internal constant reserveFactorMaxMantissa = 1e18;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L56) 
- [```uint256 internal initialExchangeRateMantissa;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L69) 
- [```mapping(address => uint256) internal accountTokens;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L107) 
- [```mapping(address => mapping(address => uint256)) internal transferAllowances;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L110) 
- [```mapping(address => BorrowSnapshot) internal accountBorrows;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L113) 
- [```uint256 internal constant expScale = 1e18;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L20) 
- [```uint256 internal constant doubleScale = 1e36;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L21) 
- [```uint256 internal constant halfExpScale = expScale / 2;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22) 
- [```uint256 internal constant mantissaOne = expScale;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L23) 
- [```function truncate(Exp memory exp) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L29) 
- [```function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L38) 
- [```function mul_ScalarTruncateAddUInt(```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L47) 
- [```function lessThanExp(Exp memory left, Exp memory right) internal pure returns (bool) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L59) 
- [```function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L63) 
- [```function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L68) 
- [```function add_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L73) 
- [```function add_(Double memory a, Double memory b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L77) 
- [```function add_(uint256 a, uint256 b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L81) 
- [```function sub_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L85) 
- [```function sub_(Double memory a, Double memory b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L89) 
- [```function sub_(uint256 a, uint256 b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L93) 
- [```function mul_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L97) 
- [```function mul_(Exp memory a, uint256 b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L101) 
- [```function mul_(uint256 a, Exp memory b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L105) 
- [```function mul_(Double memory a, Double memory b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L109) 
- [```function mul_(Double memory a, uint256 b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L113) 
- [```function mul_(uint256 a, Double memory b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L117) 
- [```function mul_(uint256 a, uint256 b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L121) 
- [```function div_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L125) 
- [```function div_(Exp memory a, uint256 b) internal pure returns (Exp memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L129) 
- [```function div_(uint256 a, Exp memory b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L133) 
- [```function div_(Double memory a, Double memory b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L137) 
- [```function div_(Double memory a, uint256 b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L141) 
- [```function div_(uint256 a, Double memory b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L145) 
- [```function div_(uint256 a, uint256 b) internal pure returns (uint256) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L149) 
- [```function fraction(uint256 a, uint256 b) internal pure returns (Double memory) {```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L153) 
- [```uint256 private constant BASE = 1e18;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L13) 
- [```RewardsDistributor[] internal rewardsDistributors;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L98) 
- [```mapping(address => bool) internal rewardsDistributorExists;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L101) 
- [```uint256 internal constant NO_ERROR = 0;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103) 
- [```uint256 internal constant closeFactorMinMantissa = 0.05e18; // 0.05```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L106) 
- [```uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L109) 
- [```uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L112) 
- [```address private protocolIncome;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L15) 
- [```address private riskFund;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L16) 
- [```uint256 private constant protocolSharePercentage = 70;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L18) 
- [```uint256 private constant baseUnit = 100;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L19) 
- [```uint256 private constant BASE = 1e18;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L12) 
- [```mapping(address => uint256) internal assetsReserves;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L13) 
- [```mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L17) 
- [```address internal poolRegistry;```](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L20) 

##### Recommendation
Add leading underscores where needed.