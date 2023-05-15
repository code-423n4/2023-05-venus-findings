## Non-Critical Issues ##
### [N-1]: Variables names are inconsistent
**Context:**

1. ```emit BadDebtRecovered(badDebtOld, badDebtNew);``` [L496](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L496) 

**Description:**

in all contracts more often "old" and "new" placed at the beginning of variable names.
Examples:
1. ```emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);``` [L791](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L791)
1. ```emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);``` [L917](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L917)
1. ```emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);``` [L319](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L319)

**Recommendation:**

Change variable name of "badDebtOld" and "badDebtNew" to "oldBadDebt" and "newBadDebt".

### [N-2]: Move validation statements to the top of the function when validating input parameters
**Context:**

1. ```require(newComptroller.isComptroller(), "marker method returned false");``` [L1141](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1141)

### [N-3]: Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
**Context:**

1. ```uint256 private constant MAX_BPS = 10000;``` [L60](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L60) 
1. ```incentiveBps = 1000;``` [L149](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L149) 

**Description:**

Scientific notation should be used for better code readability.

### [N-4]: No same value input check
**Context:**

1. ```function setCloseFactor(uint256 newCloseFactorMantissa) external {``` [L702](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L702) 
1. ```function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {``` [L779](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L779) 
1. ```function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {``` [L912](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L912) 
1. ```function setPriceOracle(PriceOracle newOracle) external onlyOwner {``` [L961](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961) 
1. ```function _setComptroller(ComptrollerInterface newComptroller) internal {``` [L1138](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1138) 
1. ```function _setShortfallContract(address shortfall_) internal {``` [L1398](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1398) 
1. ```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {``` [L1407](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1407) 
1. ```function _setShortfallContract(Shortfall shortfall_) internal {``` [L421](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L421) 
1. ```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {``` [L430](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L430) 
1. ```function setPoolRegistry(address _poolRegistry) external onlyOwner {``` [L99](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99) 
1. ```function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {``` [L126](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126) 
1. ```function setMinAmountToConvert(uint256 minAmountToConvert_) external {``` [L137](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L137) 
1. ```function setPoolRegistry(address _poolRegistry) external onlyOwner {``` [L53](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L53) 
1. ```function _setMaxLoopsLimit(uint256 limit) internal {``` [L25](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25) 

**Recommendation:**

Example how to fix ```require(_newOwner != owner, " Same address");```

### [N-5]: Add a timelock to critical functions
**Context:**

1. ```function setCloseFactor(uint256 newCloseFactorMantissa) external {``` [L702](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L702) 
1. ```function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {``` [L779](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L779) 
1. ```function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {``` [L912](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L912) 
1. ```function setPriceOracle(PriceOracle newOracle) external onlyOwner {``` [L961](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961) 
1. ```function _setComptroller(ComptrollerInterface newComptroller) internal {``` [L1138](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1138) 
1. ```function _setShortfallContract(address shortfall_) internal {``` [L1398](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1398) 
1. ```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {``` [L1407](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1407) 
1. ```function _setShortfallContract(Shortfall shortfall_) internal {``` [L421](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L421) 
1. ```function _setProtocolShareReserve(address payable protocolShareReserve_) internal {``` [L430](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L430) 
1. ```function setPoolRegistry(address _poolRegistry) external onlyOwner {``` [L99](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99) 
1. ```function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {``` [L126](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126) 
1. ```function setMinAmountToConvert(uint256 minAmountToConvert_) external {``` [L137](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L137) 
1. ```function setPoolRegistry(address _poolRegistry) external onlyOwner {``` [L53](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L53) 
1. ```function _setMaxLoopsLimit(uint256 limit) internal {``` [L25](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25) 

**Description:**

It is a good practice to give time for users to react and adjust to critical changes. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

### [N-6]: Change imports 
**Context:**

All contracts.

**Recommendation:**

Example:
Instead of ```import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";```

You can do your imports like this:
```import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol"```;

### [N-7]: Missing leading underscores
**Context:**

1. ```function updateMarketBorrowIndex(``` [L453](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L453) 
1. ```function updateMarketSupplyIndex(``` [L475](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L475) 
1. ```function calculateBorrowerReward(``` [L495](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L495) 
1. ```function calculateSupplierReward(``` [L516](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L516) 
1. ```IRiskFund private riskFund;``` [L51](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L51) 
1. ```uint256 private incentiveBps;``` [L57](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L57) 
1. ```uint256 private constant MAX_BPS = 10000;``` [L60](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L60) 
1. ```Comptroller private comptroller;``` [L52](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L52) 
1. ```address private pancakeSwapRouter;``` [L29](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L29) 
1. ```uint256 private minAmountToConvert;``` [L30](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L30) 
1. ```address private convertibleBaseAsset;``` [L31](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L31) 
1. ```address private shortfall;``` [L32](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L32) 
1. ```uint256 internal constant borrowRateMaxMantissa = 0.0005e16;``` [L53](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L53) 
1. ```uint256 internal constant reserveFactorMaxMantissa = 1e18;``` [L56](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L56) 
1. ```uint256 internal initialExchangeRateMantissa;``` [L69](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L69) 
1. ```mapping(address => uint256) internal accountTokens;``` [L107](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L107) 
1. ```mapping(address => mapping(address => uint256)) internal transferAllowances;``` [L110](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L110) 
1. ```mapping(address => BorrowSnapshot) internal accountBorrows;``` [L113](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L113) 
1. ```uint256 internal constant expScale = 1e18;``` [L20](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L20) 
1. ```uint256 internal constant doubleScale = 1e36;``` [L21](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L21) 
1. ```uint256 internal constant halfExpScale = expScale / 2;``` [L22](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22) 
1. ```uint256 internal constant mantissaOne = expScale;``` [L23](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L23) 
1. ```function truncate(Exp memory exp) internal pure returns (uint256) {``` [L29](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L29) 
1. ```function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {``` [L38](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L38) 
1. ```function mul_ScalarTruncateAddUInt(``` [L47](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L47) 
1. ```function lessThanExp(Exp memory left, Exp memory right) internal pure returns (bool) {``` [L59](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L59) 
1. ```function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {``` [L63](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L63) 
1. ```function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {``` [L68](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L68) 
1. ```function add_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {``` [L73](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L73) 
1. ```function add_(Double memory a, Double memory b) internal pure returns (Double memory) {``` [L77](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L77) 
1. ```function add_(uint256 a, uint256 b) internal pure returns (uint256) {``` [L81](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L81) 
1. ```function sub_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {``` [L85](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L85) 
1. ```function sub_(Double memory a, Double memory b) internal pure returns (Double memory) {``` [L89](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L89) 
1. ```function sub_(uint256 a, uint256 b) internal pure returns (uint256) {``` [L93](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L93) 
1. ```function mul_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {``` [L97](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L97) 
1. ```function mul_(Exp memory a, uint256 b) internal pure returns (Exp memory) {``` [L101](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L101) 
1. ```function mul_(uint256 a, Exp memory b) internal pure returns (uint256) {``` [L105](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L105) 
1. ```function mul_(Double memory a, Double memory b) internal pure returns (Double memory) {``` [L109](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L109) 
1. ```function mul_(Double memory a, uint256 b) internal pure returns (Double memory) {``` [L113](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L113) 
1. ```function mul_(uint256 a, Double memory b) internal pure returns (uint256) {``` [L117](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L117) 
1. ```function mul_(uint256 a, uint256 b) internal pure returns (uint256) {``` [L121](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L121) 
1. ```function div_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {``` [L125](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L125) 
1. ```function div_(Exp memory a, uint256 b) internal pure returns (Exp memory) {``` [L129](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L129) 
1. ```function div_(uint256 a, Exp memory b) internal pure returns (uint256) {``` [L133](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L133) 
1. ```function div_(Double memory a, Double memory b) internal pure returns (Double memory) {``` [L137](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L137) 
1. ```function div_(Double memory a, uint256 b) internal pure returns (Double memory) {``` [L141](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L141) 
1. ```function div_(uint256 a, Double memory b) internal pure returns (uint256) {``` [L145](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L145) 
1. ```function div_(uint256 a, uint256 b) internal pure returns (uint256) {``` [L149](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L149) 
1. ```function fraction(uint256 a, uint256 b) internal pure returns (Double memory) {``` [L153](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L153) 
1. ```uint256 private constant BASE = 1e18;``` [L13](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L13) 
1. ```RewardsDistributor[] internal rewardsDistributors;``` [L98](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L98) 
1. ```mapping(address => bool) internal rewardsDistributorExists;``` [L101](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L101) 
1. ```uint256 internal constant NO_ERROR = 0;``` [L103](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103) 
1. ```uint256 internal constant closeFactorMinMantissa = 0.05e18; // 0.05``` [L106](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L106) 
1. ```uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9``` [L109](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L109) 
1. ```uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9``` [L112](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L112) 
1. ```address private protocolIncome;``` [L15](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L15) 
1. ```address private riskFund;``` [L16](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L16) 
1. ```uint256 private constant protocolSharePercentage = 70;``` [L18](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L18) 
1. ```uint256 private constant baseUnit = 100;``` [L19](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L19) 
1. ```uint256 private constant BASE = 1e18;``` [L12](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L12) 
1. ```mapping(address => uint256) internal assetsReserves;``` [L13](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L13) 
1. ```mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;``` [L17](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L17) 
1. ```address internal poolRegistry;``` [L20](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L20) 

**Description:**

Internal and private functions, state variables, constants, and immutables should starting with an underscore.

### [N-8]: Emit events in initialize() functions
**Context:**

1. ```function initialize(uint256 loopLimit, address accessControlManager) external initializer {``` [L138](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L138) 
1. ```function initialize(``` [L59](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L59) 
1. ```function initialize(``` [L131](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L131) 
1. ```function initialize(``` [L111](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L111) 
1. ```function initialize(``` [L164](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164) 
1. ```function initialize(``` [L73](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73) 
1. ```function initialize(address _protocolIncome, address _riskFund) external initializer {``` [L39](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L39) 

**Recommendation:**

Define an initialize event and emit that at the end of  initialize() function.

### [N-9]: Event is never emitted
**Context:**

1. ```event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);``` [L47](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L47) 

**Description:**

There is the event defined that is never emitted and it can be removed.
