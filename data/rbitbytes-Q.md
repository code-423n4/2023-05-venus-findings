Missing Event for critical parameters init and change

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L127-L132

constructor(address poolRegistry_) {
    require(poolRegistry_ != address(0), "invalid pool registry address");

    poolRegistry = poolRegistry_;
    _disableInitializers();
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73-L93

function initialize(
    address pancakeSwapRouter_,
    uint256 minAmountToConvert_,
    address convertibleBaseAsset_,
    address accessControlManager_,
    uint256 loopsLimit_
) external initializer {
    require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
    require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
    require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

    __Ownable2Step_init();
    __AccessControlled_init_unchained(accessControlManager_);

    pancakeSwapRouter = pancakeSwapRouter_;
    minAmountToConvert = minAmountToConvert_;
    convertibleBaseAsset = convertibleBaseAsset_;

    _setMaxLoopsLimit(loopsLimit_);
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L65-L77

constructor(
    uint256 baseRatePerYear,
    uint256 multiplierPerYear,
    uint256 jumpMultiplierPerYear,
    uint256 kink_,
    IAccessControlManagerV8 accessControlManager_
) {
    require(address(accessControlManager_) != address(0), "invalid ACM address");

    accessControlManager = accessControlManager_;

    _updateJumpRateModel(baseRatePerYear, multiplierPerYear, jumpMultiplierPerYear, kink_);
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73-L93

function initialize(
    address pancakeSwapRouter_,
    uint256 minAmountToConvert_,
    address convertibleBaseAsset_,
    address accessControlManager_,
    uint256 loopsLimit_
) external initializer {
    require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
    require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
    require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

    __Ownable2Step_init();
    __AccessControlled_init_unchained(accessControlManager_);

    pancakeSwapRouter = pancakeSwapRouter_;
    minAmountToConvert = minAmountToConvert_;
    convertibleBaseAsset = convertibleBaseAsset_;

    _setMaxLoopsLimit(loopsLimit_);
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73-L93

function initialize(
    address pancakeSwapRouter_,
    uint256 minAmountToConvert_,
    address convertibleBaseAsset_,
    address accessControlManager_,
    uint256 loopsLimit_
) external initializer {
    require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
    require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
    require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

    __Ownable2Step_init();
    __AccessControlled_init_unchained(accessControlManager_);

    pancakeSwapRouter = pancakeSwapRouter_;
    minAmountToConvert = minAmountToConvert_;
    convertibleBaseAsset = convertibleBaseAsset_;

    _setMaxLoopsLimit(loopsLimit_);
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L11-L23

constructor(
    uint256 baseRatePerYear,
    uint256 multiplierPerYear,
    uint256 jumpMultiplierPerYear,
    uint256 kink_,
    IAccessControlManagerV8 accessControlManager_
)
    BaseJumpRateModelV2(baseRatePerYear, multiplierPerYear, jumpMultiplierPerYear, kink_, accessControlManager_)
/* solhint-disable-next-line no-empty-blocks */
{

}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73-L93

function initialize(
    address pancakeSwapRouter_,
    uint256 minAmountToConvert_,
    address convertibleBaseAsset_,
    address accessControlManager_,
    uint256 loopsLimit_
) external initializer {
    require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
    require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
    require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

    __Ownable2Step_init();
    __AccessControlled_init_unchained(accessControlManager_);

    pancakeSwapRouter = pancakeSwapRouter_;
    minAmountToConvert = minAmountToConvert_;
    convertibleBaseAsset = convertibleBaseAsset_;

    _setMaxLoopsLimit(loopsLimit_);
}


https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L7-L9

constructor(address implementation_) UpgradeableBeacon(implementation_) {
    require(implementation_ != address(0), "Invalid implementation address");
}


Generate perfect code headers every time

I suggest using header for Solidity code layout and readability:

https://github.com/transmissions11/headers

/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/