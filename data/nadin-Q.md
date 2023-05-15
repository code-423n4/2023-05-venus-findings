# QA Report
## Summary
Total 03 Low and 10 Non-Critical
### Low Risk Issues
- [L-01]  `RiskFund.sol#_swapAsset()` uses the wrong function for swapping fee-on-transfer tokens
- [L-02] `IPancakeswapV2Router.sol#swapExactTokensForTokens` does not follow the standard
- [L-03] Check the condition for the `_incentiveBps` argument in `Shortfall.sol#updateIncentiveBps()`
### Non-Critical Issues
- [N-01] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
- [N-02] Inefficient Event Arguments
- [N-03] Large numeric literals should use underscores for readability (missed by bot)
- [N-04] immutable should be UPPERCASE
- [N-05] Use SMTChecker
- [N-06] Take advantage of Custom Error’s return value property
- [N-07] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
- [N-08] Use a single file for all system-wide constants
- [N-09] Constant redefined elsewhere (Missed by bot)
- [N-10] Use named parameters for mapping type declarations

## [L-01]  `RiskFund.sol#_swapAsset()` uses the wrong function for swapping fee-on-transfer tokens
### Impact
Number of swapped tokens may be lower than actual when the underlying is a fee-on-transfer token with the fee turned on. The function uses `swapExactTokensForTokens()` when it should use `swapExactTokensForTokensSupportingFeeOnTransferTokens()` instead.
### Recommendation
Use [swapExactTokensForTokensSupportingFeeOnTransferTokens()](https://docs.pancakeswap.finance/code/smart-contracts/pancakeswap-exchange/v2/router-v2#swapexacttokensforethsupportingfeeontransfertokens)

## [L-02] `IPancakeswapV2Router.sol#swapExactTokensForTokens` does not follow the standard
```
File: IPancakeswapV2Router.sol
interface IPancakeswapV2Router {
    null(
        uint256 amountIn,
        uint256 amountOutMin,
        address[] calldata path,
        address to,
        uint256 deadline
    ) external returns (uint256[] memory amounts);
```
[It's](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/IPancakeswapV2Router.sol#L4-L11) called in [RiskFund#_swappAsset](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L260)
```
File: RiskFund.sol
                    uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
```
According to [docs](https://docs.pancakeswap.finance/code/smart-contracts/pancakeswap-exchange/v2/router-v2#swapexacttokensfortokens) the parameters `amountIn, amountOutMin, deadline` are `uint` not `uint256`
### Recommnendation
Comply with the standards of pancakeswap

## [L-03] Check the condition for the `_incentiveBps` argument in `Shortfall.sol#updateIncentiveBps()`
Add condition `_incentiveBps` < MAX_BPS (10_000)
```
307:    null(uint256 _incentiveBps) external {
308:        _checkAccessAllowed("updateIncentiveBps(uint256)");
- 309:        require(_incentiveBps != 0, "incentiveBps must not be 0");
+ 309:       require(_incentiveBps != 0 && ­_incentiveBps < MAX_BPS, " Invalid incentiveBps ");
310:        uint256 oldIncentiveBps = incentiveBps;
311:        incentiveBps = _incentiveBps;
312:        emit IncentiveBpsUpdated(oldIncentiveBps, _incentiveBps);
313:    }
```

## [N-01] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
```
File: Shortfall.sol
44:        mapping(VToken => uint256) marketDebt;
72:    mapping(address => Auction) public auctions;
```
[hortfall.sol#L44](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L44)
```
File: RewardsDistributor.sol
23:    mapping(address => RewardToken) public rewardTokenSupplyState;
26:    mapping(address => mapping(address => uint256)) public rewardTokenSupplierIndex;
32:    mapping(address => uint256) public rewardTokenAccrued;
35:    mapping(address => uint256) public rewardTokenBorrowSpeeds;
38:    mapping(address => uint256) public rewardTokenSupplySpeeds;
41:    mapping(address => RewardToken) public rewardTokenBorrowState;
44:    mapping(address => uint256) public rewardTokenContributorSpeeds;
47:    mapping(address => uint256) public lastContributorBlock;
50:    mapping(address => mapping(address => uint256)) public rewardTokenBorrowerIndex;
```
[RewardsDistributor.sol#L50](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L50)
```
File: PoolRegistry.sol
83:    mapping(address => VenusPoolMetaData) public metadata;
88:    mapping(uint256 => address) private _poolsByID;
98:    mapping(address => VenusPool) private _poolByComptroller;
103:    mapping(address => mapping(address => address)) private _vTokens;
108:    mapping(address => address[]) private _supportedPools;
```
[PoolRegistry.sol#L108](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L108)
```
File: RiskFund.sol
35:    mapping(address => uint256) public poolReserves;
```
[RiskFund.sol#L35](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L35)
```
File: VTokenInterfaces.sol
107:    mapping(address => uint256) internal accountTokens;
110:    mapping(address => mapping(address => uint256)) internal transferAllowances;
113:    mapping(address => BorrowSnapshot) internal accountBorrows;
```
[VTokenInterfaces.sol#L113](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VTokenInterfaces.sol#L113)
```
File: ComptrollerStorage.sol
41:        mapping(address => bool) accountMembership;
74:    mapping(address => VToken[]) public accountAssets;
80:    mapping(address => Market) public markets;
86:    mapping(address => uint256) public borrowCaps;
92:    mapping(address => uint256) public supplyCaps;
95:    mapping(address => mapping(Action => bool)) internal _actionPaused;
101:    mapping(address => bool) internal rewardsDistributorExists;
```
[ComptrollerStorage.sol#L101](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L101)
```
File: ReserveHelpers.sol
13:    mapping(address => uint256) internal assetsReserves;
17:    mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;
```
[ReserveHelpers.sol#L17](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L17)

## [N-02] Inefficient Event Arguments
### Description:
The linked event is meant to emit the previous and new value of a storage variable being
adjusted, however, to do so it redundantly uses a local variable.
### Contexts:
```
File: Comptroller.sol
709:        emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
791:        emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);
917:        emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);
966:        emit NewPriceOracle(oldOracle, newOracle);
```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```
File: VToken.sol
319:        emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);
```
[VToken.sol#319](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L319)

```
File: PoolRegistry.sol
337:        emit PoolNameSet(comptroller, oldName, name);
```
[PoolRegistry.sol#337](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L337)

```
File: RiskFund/ProtocolShareReserve.sol
57:        emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
```
[ProtocolShareReserve.sol#57](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L57)

```
File: RiskFund/RiskFund.sol
103:        emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
119:        emit ShortfallContractUpdated(oldShortfallContractAddress, shortfallContractAddress_);
130:        emit PancakeSwapRouterUpdated(oldPancakeSwapRouter, pancakeSwapRouter_);
142:        emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
```
[RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)
### Recommendation:
Advise the emission to occur prior to the assignment of the storage variable by setting
the first argument of the event as the existing storage value and the second argument as the
input argument of the function.


## [N-03] Large numeric literals should use underscores for readability (missed by bot)
```
File: Shortfall.sol
- 60:    uint256 private constant MAX_BPS = 10000;
+ 60:    uint256 private constant MAX_BPS = 10_000;
```
[Shortfall.sol#60](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L60)

## [N-04] immutable should be UPPERCASE
```
File: Comptroller.sol
27:    address public immutable poolRegistry;
```
[Comptroller.sol#L27](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L27)
```
File: WhitePaperInterestRateModel.sol
22:    uint256 public immutable multiplierPerBlock;
27:    uint256 public immutable baseRatePerBlock;
```
[WhitePaperInterestRateModel.sol#L22-L27](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L22-L27)

## [N-05] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-06] Take advantage of Custom Error’s return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly
### Contexts:
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)
[ErrorReporter.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol)

## [N-07] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
### Recommendation: Use form like FraxFinance project
https://github.com/FraxFinance/frxETH-public/blob/7f7731dbc93154131aba6e741b6116da05b25662/src/sfrxETH.sol#L4-L24

## [N-08] Use a single file for all system-wide constants
### Description:
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

#### constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## [N-09] Constant redefined elsewhere (Missed by bot)
### Contexts:
```
File: ComptrollerStorage.sol
103:    uint256 internal constant NO_ERROR = 0;
File: ErrorReporter.sol
5:    uint256 public constant NO_ERROR = 0; // support legacy return codes
```
[ComptrollerStorage.sol#L103](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L103) vs [ErrorReporter.sol#L5](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol#L5)

```
File: ComptrollerStorage.sol
109:    uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9
112:    uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9
```
[ComptrollerStorage.sol#L109-L112](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L109-L112)

```
File: BaseJumpRateModelV2.sol
13:    uint256 private constant BASE = 1e18;
File: ExponentialNoError.sol
20:    uint256 internal constant expScale = 1e18;
File: VTokenInterfaces.sol
56:    uint256 internal constant reserveFactorMaxMantissa = 1e18;
File: WhitePaperInterestRateModel.sol
12:    uint256 private constant BASE = 1e18;
```
[BaseJumpRateModelV2.sol#L13](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L13) vs [ExponentialNoError.sol#L20](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ExponentialNoError.sol#L20) vs [VTokenInterfaces.sol#L56](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VTokenInterfaces.sol#L56) vs [WhitePaperInterestRateModel.sol#L12](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L12)

## [N-10] Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18
### Contexts:
Cases already listed in `N-01`