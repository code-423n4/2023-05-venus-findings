# QA Report
## Issue List
| Number |Issues Details|Instances|
|:--:|:-------|:--:|
|[L-01]|Add Event-Emit for State Changes| 2 |
|[L-02]|Missing Event for initialize| 6 |
|[L-03]|There is an inconsistency in ```blocksPerYear``` between contracts that can cause conflict. | 1 |
|[NC-01]|Named return variables are not used inside function| 4 |
|[NC-02]|Immutables should follow the naming convention ```UPPERCASE_WITH_UNDERSCORES```| 3 |
|[NC-03]|Function natspec and declaration property does not match| 1 |
|[NC-04]|Initial value check is missing in Set Functions| 5 |
|[NC-05]|For functions, follow Solidity standard naming conventions| 15 |
|[NC-06]|Use checks (require) before action, not after| 1 |
|[NC-07]|Constants should be named with all capital letters with underscores separating words.| 17 |


Total 10 issues with 55 instances

## Findings

### [L-01] Add Event-Emit for State Changes

A recommended guideline for using events in Solidity is to record them whenever there is a change in the contract's state.


```solidity
venus/contracts/Comptroller.sol

  function healAccount(address user) external {
        VToken[] memory userAssets = accountAssets[user];
        uint256 userAssetsCount = userAssets.length;

        address liquidator = msg.sender;
        // We need all user's markets to be fresh for the computations to be correct
        for (uint256 i; i < userAssetsCount; ++i) {
            userAssets[i].accrueInterest();
            oracle.updatePrice(address(userAssets[i]));
        }

        AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(user, _getLiquidationThreshold);

        if (snapshot.totalCollateral > minLiquidatableCollateral) {
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }

        if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }

        // percentage = collateral / (borrows * liquidation incentive)
        Exp memory collateral = Exp({ mantissa: snapshot.totalCollateral });
        Exp memory scaledBorrows = mul_(
            Exp({ mantissa: snapshot.borrows }),
            Exp({ mantissa: liquidationIncentiveMantissa })
        );

        Exp memory percentage = div_(collateral, scaledBorrows);
        if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
            revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
        }

        for (uint256 i; i < userAssetsCount; ++i) {
            VToken market = userAssets[i];

            (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);
            uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);

            // Seize the entire collateral
            if (tokens != 0) {
                market.seize(liquidator, user, tokens);
            }
            // Repay a certain percentage of the borrow, forgive the rest
            if (borrowBalance != 0) {
                market.healBorrow(liquidator, user, repaymentAmount);
            }
        }
    }
    
    ...
    
     function liquidateAccount(address borrower, LiquidationOrder[] calldata orders) external {
        // We will accrue interest and update the oracle prices later during the liquidation

        AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(borrower, _getLiquidationThreshold);

        if (snapshot.totalCollateral > minLiquidatableCollateral) {
            // You should use the regular vToken.liquidateBorrow(...) call
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }

        uint256 collateralToSeize = mul_ScalarTruncate(
            Exp({ mantissa: liquidationIncentiveMantissa }),
            snapshot.borrows
        );
        if (collateralToSeize >= snapshot.totalCollateral) {
            // There is not enough collateral to seize. Use healBorrow to repay some part of the borrow
            // and record bad debt.
            revert InsufficientCollateral(collateralToSeize, snapshot.totalCollateral);
        }

        if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }

        uint256 ordersCount = orders.length;

        _ensureMaxLoops(ordersCount);

        for (uint256 i; i < ordersCount; ++i) {
            if (!markets[address(orders[i].vTokenBorrowed)].isListed) {
                revert MarketNotListed(address(orders[i].vTokenBorrowed));
            }
            if (!markets[address(orders[i].vTokenCollateral)].isListed) {
                revert MarketNotListed(address(orders[i].vTokenCollateral));
            }

            LiquidationOrder calldata order = orders[i];
            order.vTokenBorrowed.forceLiquidateBorrow(
                msg.sender,
                borrower,
                order.repayAmount,
                order.vTokenCollateral,
                true
            );
        }

        VToken[] memory borrowMarkets = accountAssets[borrower];
        uint256 marketsCount = borrowMarkets.length;

        for (uint256 i; i < marketsCount; ++i) {
            (, uint256 borrowBalance, ) = _safeGetAccountSnapshot(borrowMarkets[i], borrower);
            require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
        }
    }

```
1 file 2 instances 


### [L-02] Missing Event for initialize

In Solidity, events serve two important purposes: they enable external tools to track changes made to the contract and they help users stay informed about any changes that occur. It's recommended to emit events during initialization, as this is a step that many projects tend to overlook. By doing so, it ensures that any changes made during initialization can also be tracked and that users won't be caught off-guard by unexpected modifications.
Although ```__Ownable2Step_init()``` has event "OwnershipTransferStarted" in itself, it doesn't automatically emit event. It has to be done manually.

```solidity
venus/contracts/Comptroller.sol

function initialize(uint256 loopLimit, address accessControlManager) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager);

        _setMaxLoopsLimit(loopLimit);
    }
```

```solidity
venus/contracts/VToken.sol

  function initialize(
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
        require(admin_ != address(0), "invalid admin address");

        // Initialize the market
        _initialize(
            underlying_,
            comptroller_,
            interestRateModel_,
            initialExchangeRateMantissa_,
            name_,
            symbol_,
            decimals_,
            admin_,
            accessControlManager_,
            riskManagement,
            reserveFactorMantissa_
        );
    }
```
```solidity
venus/contracts/Shortfall/Shortfall.sol

function initialize(
        address convertibleBaseAsset_,
        IRiskFund riskFund_,
        uint256 minimumPoolBadDebt_,
        address accessControlManager_
    ) external initializer {
        require(convertibleBaseAsset_ != address(0), "invalid base asset address");
        require(address(riskFund_) != address(0), "invalid risk fund address");
        require(minimumPoolBadDebt_ != 0, "invalid minimum pool bad debt");

        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager_);
        __ReentrancyGuard_init();
        minimumPoolBadDebt = minimumPoolBadDebt_;
        convertibleBaseAsset = convertibleBaseAsset_;
        riskFund = riskFund_;
        waitForFirstBidder = 100;
        nextBidderBlockLimit = 10;
        incentiveBps = 1000;
    }
    
```
```solidity
venus/contracts/Rewards/RewardsDistributor.sol

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

```solidity
venus/contracts/Pool/PoolRegistry.sol

 function initialize(
        VTokenProxyFactory vTokenFactory_,
        JumpRateModelFactory jumpRateFactory_,
        WhitePaperInterestRateModelFactory whitePaperFactory_,
        Shortfall shortfall_,
        address payable protocolShareReserve_,
        address accessControlManager_
    ) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager_);

        vTokenFactory = vTokenFactory_;
        jumpRateFactory = jumpRateFactory_;
        whitePaperFactory = whitePaperFactory_;
        _setShortfallContract(shortfall_);
        _setProtocolShareReserve(protocolShareReserve_);
    }
```
```solidity
venus/contracts/RiskFund/ProtocolShareReserve.sol

function initialize(address _protocolIncome, address _riskFund) external initializer {
        require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
        require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

        __Ownable2Step_init();

        protocolIncome = _protocolIncome;
        riskFund = _riskFund;
    }
```
6 files 6 issues

### [L-03] There is an inconsistency in ```blocksPerYear``` between contracts that can cause conflict.

In the "WhitePaperInterestRateModel" contract there is a constant "blocksPerYear" defined as "2102400" while the same constant in
"BaseJumpRateModelV2" contract is defined as "10512000". According to "BaseJumpRateModelV2" contracts NatSpec comments it states that
the contract is taken away from Compund's "JumpRateModelV2" contract which same constant defined as "2102400".

```solidity
venus/contracts/BaseJumpRateModelV2.sol

 /**
     * @notice The approximate number of blocks per year that is assumed by the interest rate model
     */
    uint256 public constant blocksPerYear = 10512000;
```
```solidity
venus/contracts/WhitePaperInterestRateModel.sol

 /**
     * @notice The approximate number of blocks per year that is assumed by the interest rate model
     */
    uint256 public constant blocksPerYear = 2102400;
```
2 files 1 issue


### [NC-01] Named return variables are not used inside function
To enhance the readability and clarity of code and to minimize the likelihood of regressions during future code refactoring, it may be advantageous to establish a standardized approach to return values throughout the codebase. This can be accomplished by eliminating named return variables, explicitly defining them as local variables, and adding appropriate return statements
Therefore, it is recommended to consider adopting a consistent approach to handling return values in Solidity code.

```solidity
venus/contracts/Comptroller.sol

function getAccountLiquidity(address account)
        external
        view
        returns (
            uint256 error,
            uint256 liquidity,
            uint256 shortfall
        )
    {
        AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(account, _getCollateralFactor);
        return (NO_ERROR, snapshot.liquidity, snapshot.shortfall);
    }

...

function getHypotheticalAccountLiquidity(
        address account,
        address vTokenModify,
        uint256 redeemTokens,
        uint256 borrowAmount
    )
        external
        view
        returns (
            uint256 error,
            uint256 liquidity,
            uint256 shortfall
        )
    {
        AccountLiquiditySnapshot memory snapshot = _getHypotheticalLiquiditySnapshot(
            account,
            VToken(vTokenModify),
            redeemTokens,
            borrowAmount,
            _getCollateralFactor
        );
        return (NO_ERROR, snapshot.liquidity, snapshot.shortfall);
    }

...

function liquidateCalculateSeizeTokens(
        address vTokenBorrowed,
        address vTokenCollateral,
        uint256 actualRepayAmount
    ) external view override returns (uint256 error, uint256 tokensToSeize) {
        /* Read oracle prices for borrowed and collateral markets */
        uint256 priceBorrowedMantissa = _safeGetUnderlyingPrice(VToken(vTokenBorrowed));
        uint256 priceCollateralMantissa = _safeGetUnderlyingPrice(VToken(vTokenCollateral));

        /*
         * Get the exchange rate and calculate the number of collateral tokens to seize:
         *  seizeAmount = actualRepayAmount * liquidationIncentive * priceBorrowed / priceCollateral
         *  seizeTokens = seizeAmount / exchangeRate
         *   = actualRepayAmount * (liquidationIncentive * priceBorrowed) / (priceCollateral * exchangeRate)
         */
        uint256 exchangeRateMantissa = VToken(vTokenCollateral).exchangeRateStored(); // Note: reverts on error
        uint256 seizeTokens;
        Exp memory numerator;
        Exp memory denominator;
        Exp memory ratio;

        numerator = mul_(Exp({ mantissa: liquidationIncentiveMantissa }), Exp({ mantissa: priceBorrowedMantissa }));
        denominator = mul_(Exp({ mantissa: priceCollateralMantissa }), Exp({ mantissa: exchangeRateMantissa }));
        ratio = div_(numerator, denominator);

        seizeTokens = mul_ScalarTruncate(ratio, actualRepayAmount);

        return (NO_ERROR, seizeTokens);
    }
```

```solidity
venus/contracts/Pool/PoolRegistry.sol

 function createRegistryPool(
        string calldata name,
        address beaconAddress,
        uint256 closeFactor,
        uint256 liquidationIncentive,
        uint256 minLiquidatableCollateral,
        address priceOracle,
        uint256 maxLoopsLimit,
        address accessControlManager
    ) external virtual returns (uint256 index, address proxyAddress) {
        _checkAccessAllowed("createRegistryPool(string,address,uint256,uint256,uint256,address,uint256,address)");
        // Input validation
        require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");
        require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

        BeaconProxy proxy = new BeaconProxy(
            beaconAddress,
            abi.encodeWithSelector(Comptroller.initialize.selector, maxLoopsLimit, accessControlManager)
        );

        proxyAddress = address(proxy);
        Comptroller comptrollerProxy = Comptroller(proxyAddress);

        uint256 poolId = _registerPool(name, proxyAddress);

        // Set Venus pool parameters
        comptrollerProxy.setCloseFactor(closeFactor);
        comptrollerProxy.setLiquidationIncentive(liquidationIncentive);
        comptrollerProxy.setMinLiquidatableCollateral(minLiquidatableCollateral);
        comptrollerProxy.setPriceOracle(PriceOracle(priceOracle));

        // Start transferring ownership to msg.sender
        comptrollerProxy.transferOwnership(msg.sender);

        // Register the pool with this PoolRegistry
        return (poolId, proxyAddress);
    }
```
2 files 4 issues

### [NC-02] Immutables should follow the naming convention ```UPPERCASE_WITH_UNDERSCORES```

```solidity
venus/contracts/Comptroller.sol

    address public immutable poolRegistry;
```
```solidity
venus/contracts/WhitePaperInterestRateModel.sol

 /**
     * @notice The multiplier of utilization rate that gives the slope of the interest rate
     */
    uint256 public immutable multiplierPerBlock;

    /**
     * @notice The base interest rate which is the y-intercept when utilization rate is 0
     */
    uint256 public immutable baseRatePerBlock;
```
2 files 3 issues


### [NC-03] Function natspec and declaration property does not match

Function declared as external while natspec @notice explain the function as it is public function.

```solidity
venus/contracts/VToken.sol

/**
     * @notice A public function to sweep accidental ERC-20 transfers to this contract. Tokens are sent to admin (timelock)
     * @param token The address of the ERC-20 token to sweep
     * @custom:access Only Governance
     */
    function sweepToken(IERC20Upgradeable token) external override {
        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
        require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
        uint256 balance = token.balanceOf(address(this));
        token.safeTransfer(owner(), balance);

        emit SweepToken(address(token));
    }
```
1 file 1 issue

### [NC-04] Initial value check is missing in Set Functions

It is optimal to check whether the current value and new value are the same or not. 

```solidity
venus/contracts/VToken.sol

 function setInterestRateModel(InterestRateModel newInterestRateModel) external override {
        _checkAccessAllowed("setInterestRateModel(address)");

        accrueInterest();
        _setInterestRateModelFresh(newInterestRateModel);
    }

...

 function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {
        _setProtocolShareReserve(protocolShareReserve_);
    }
    
...
Ownable2StepUpgradeable
function setShortfallContract(address shortfall_) external onlyOwner {
        _setShortfallContract(shortfall_);
    }

...

function setProtocolSeizeShare(uint256 newProtocolSeizeShareMantissa_) external {
        _checkAccessAllowed("setProtocolSeizeShare(uint256)");
        uint256 liquidationIncentive = ComptrollerViewInterface(address(comptroller)).liquidationIncentiveMantissa();
        if (newProtocolSeizeShareMantissa_ + 1e18 > liquidationIncentive) {
            revert ProtocolSeizeShareTooBig();
        }

        uint256 oldProtocolSeizeShareMantissa = protocolSeizeShareMantissa;
        protocolSeizeShareMantissa = newProtocolSeizeShareMantissa_;
        emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);
    }
    
...

 function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {
        _checkAccessAllowed("setReserveFactor(uint256)");

        accrueInterest();
        _setReserveFactorFresh(newReserveFactorMantissa);
    }
```
1 file 5 issues

 ### [NC-05] For functions, follow Solidity standard naming conventions
 
 It is recommended to use ```_mixedCase``` naming convention for internal and private functions.
 
```solidity
venus/contracts/VToken.sol

 function updateMarketBorrowIndex(
        address vToken,
        RewardsDistributor rewardsDistributor,
        RewardTokenState memory borrowState,
        Exp memory marketBorrowIndex
    ) internal view {
    
...

function updateMarketSupplyIndex(
        address vToken,
        RewardsDistributor rewardsDistributor,
        RewardTokenState memory supplyState
    ) internal view {
    
...

function calculateBorrowerReward(
        address vToken,
        RewardsDistributor rewardsDistributor,
        address borrower,
        RewardTokenState memory borrowState,
        Exp memory marketBorrowIndex
    ) internal view returns (uint256) {

...

 function calculateSupplierReward(
        address vToken,
        RewardsDistributor rewardsDistributor,
        address supplier,
        RewardTokenState memory supplyState
    ) internal view returns (uint256) {

```
```solidity
venus/contracts/ExponentialNoError.sol

function truncate(Exp memory exp) internal pure returns (uint256) {
        // Note: We are not using careful math here as we're performing a division that cannot fail
        return exp.mantissa / expScale;
    }

...

function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {
        Exp memory product = mul_(a, scalar);
        return truncate(product);
    }
    
function mul_ScalarTruncateAddUInt(
        Exp memory a,
        uint256 scalar,
        uint256 addend
    ) internal pure returns (uint256) {
        Exp memory product = mul_(a, scalar);
        return add_(truncate(product), addend);
    }

    function lessThanExp(Exp memory left, Exp memory right) internal pure returns (bool) {
        return left.mantissa < right.mantissa;
    }

    function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {
        require(n < 2**224, errorMessage);
        return uint224(n);
    }

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
        require(n < 2**32, errorMessage);
        return uint32(n);
    }

    function add_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }

    function sub_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a - b;
    }

    function mul_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a * b;
    }

    function div_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a / b;
    }

    function fraction(uint256 a, uint256 b) internal pure returns (Double memory) {
        return Double({ mantissa: div_(mul_(a, doubleScale), b) });
    }
```
2 files 15 issues


### [NC-06] Use checks (require) before action, not after

In the following code require statement used after calling internal function and performing operation. Although the actions will revert 
if code can not pass require statement, it is best practice to check before action, not after.

```solidity
venus/contracts/Rewards/RewardsDistributor.sol

 function grantRewardToken(address recipient, uint256 amount) external onlyOwner {
        uint256 amountLeft = _grantRewardToken(recipient, amount);
        require(amountLeft == 0, "insufficient rewardToken for grant");
        emit RewardTokenGranted(recipient, amount);
    }
```
1 file 1 issue

### [NC-07] Constants should be named with all capital letters with underscores separating words.

It is recommended in solidity style guide to name constants with all capital letters with underscores separating words.
Examples:
MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION

```solidity
venus/contracts/Rewards/RewardsDistributor.sol

  uint224 public constant rewardTokenInitialIndex = 1e36;
```
```solidity
venus/contracts/VTokenInterfaces.sol

// Maximum borrow rate that can ever be applied (.0005% / block)
    uint256 internal constant borrowRateMaxMantissa = 0.0005e16;

    // Maximum fraction of interest that can be set aside for reserves
    uint256 internal constant reserveFactorMaxMantissa = 1e18;
    
     /**
     * @notice Indicator that this is a VToken contract (for inspection)
     */
    bool public constant isVToken = true;
```
```solidity
venus/contracts/ExponentialNoError.sol


    uint256 internal constant expScale = 1e18;
    uint256 internal constant doubleScale = 1e36;
    uint256 internal constant halfExpScale = expScale / 2;
    uint256 internal constant mantissaOne = expScale;
```
```solidity
venus/contracts/BaseJumpRateModelV2.sol

int256 public constant blocksPerYear = 10512000;
```
```solidity
venus/contracts/ComptrollerStorage.sol
// closeFactorMantissa must be strictly greater than this value
    uint256 internal constant closeFactorMinMantissa = 0.05e18; // 0.05

    // closeFactorMantissa must not exceed this value
    uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9

    // No collateralFactorMantissa may exceed this value
    uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9

    /// @notice Indicator that this is a Comptroller contract (for inspection)
    bool internal constant _isComptroller = true;
```
```solidity
venus/contracts/RiskFund/ProtocolShareReserve.sol

 uint256 private constant protocolSharePercentage = 70;
 uint256 private constant baseUnit = 100;
```
```solidity
venus/contracts/WhitePaperInterestRateModel.sol

uint256 public constant blocksPerYear = 2102400;
```
```solidity
venus/contracts/InterestRateModel.sol

bool public constant isInterestRateModel = true;
```
8 files 17 issues
