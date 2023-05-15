# gas optmization
## Summary 

|No  |  issue   | instance  |
|--- |----------|-----------|
|[G‑1]| Do not calculate constants|1|
|[G-2]|State variables can be packed to use fewer storage slots|1|
|[G-3]|Use != 0 instead of > 0 for unsigned integer comparison|13|
|[G-4]| Avoid using state variable in emit |3|
|[G-5]| Not using the named return variables when a function returns, wastes deployment gas|2|
|[G-6]|Remove the initializer modifier|
|[G-7]| Use constants instead of type(uintx).max | 5
|[G-8]|Use calldata instead of memory|2|
|[G‑9]| Use double if statements instead of && | 8|
|[G‑10]|Can Make The Variable Outside The Loop To Save Gas |10|
|[G-11]| Make 3 event parameters indexed when possible|35|
|[G-12]| Use hardcode address instead address(this)|17|
|[G-13]| Sort Solidity operations using short-circuit mode|1|






## [G-01] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
File: /main/contracts/ExponentialNoError.sol
22    uint256 internal constant halfExpScale = expScale / 2;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22



## [G-02] State variables can be packed to use fewer storage slots


The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Gas savings: 2000
```solidity
123   address public shortfall;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L123




##  [G-03]Use != 0 instead of > 0 for unsigned integer comparison

```solidity
File: /main/contracts/Lens/PoolLens.sol
462   if (deltaBlocks > 0 && borrowSpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462


```solidity

466   Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L466

```solidity
470   } else if (deltaBlocks > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L470

```solidity
483   if (deltaBlocks > 0 && supplySpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L483

```solidity
486    Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L486

```solidity
490    } else if (deltaBlocks > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L490


```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
261    if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261    

```solidity
418    if (amount > 0 && amount <= rewardTokenRemaining) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L218

```solidity
435    if (deltaBlocks > 0 && supplySpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L435

```solidity
438    Double memory ratio = supplyTokens > 0
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L438
446    } else if (deltaBlocks > 0) {

```solidity
463    if (deltaBlocks > 0 && borrowSpeed > 0) { 
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L463


## [G-4] Avoid using state variable in emit 
Using a state variable in SetOwner emits wastes gas.


```solidity
File: /main/contracts/BaseJumpRateModelV2.sol
162  emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L162

```solidity
File: /main/contracts/MaxLoopsLimitHelper.sol
31    emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L31

```solidity
File: /main/contracts/WhitePaperInterestRateModel.sol
40    emit NewInterestParams(baseRatePerBlock, multiplierPerBlock);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L40


## [G-05]Not using the named return variables when a function returns, wastes deployment gas

```solidity
File: /main/contracts/Comptroller.sol
169   return results;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L169

```silidity
1039  return allMarkets;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1039

## [G-06] Remove the initializer modifier
If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
164   function initialize(
        VTokenProxyFactory vTokenFactory_,
        JumpRateModelFactory jumpRateFactory_,
        WhitePaperInterestRateModelFactory whitePaperFactory_,
        Shortfall shortfall_,
        address payable protocolShareReserve_,
        address accessControlManager_
    ) external initializer {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
39    function initialize(address _protocolIncome, address _riskFund) external initializer {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L39


```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
111    function initialize(
        Comptroller comptroller_,
        IERC20Upgradeable rewardToken_,
        uint256 loopsLimit_,
        address accessControlManager_
    ) external initializer {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L111

```solidity
File: /main/contracts/RiskFund/RiskFund.sol
73   function initialize(
        address pancakeSwapRouter_,
        uint256 minAmountToConvert_,
        address convertibleBaseAsset_,
        address accessControlManager_,
        uint256 loopsLimit_
    ) external initializer {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L73


```solidity
File: /main/contracts/Shortfall/Shortfall.sol
131   function initialize(
        address convertibleBaseAsset_,
        IRiskFund riskFund_,
        uint256 minimumPoolBadDebt_,
        address accessControlManager_
    ) external initializer {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L131

```solidity
File: /main/contracts/VToken.sol
59   function initialize(
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
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L59

## [G-07] Use constants instead of type(uintx).max


```solidity
262    if (supplyCap != type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L262


```solidity
351    if (borrowCap != type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L351

```solidity
File: /main/contracts/VToken.sol
1055   if (repayAmount == type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1055
```solidity
1314   startingAllowance = type(uint256).max;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1314

```solidity
1331   if (startingAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1331


## [G-8] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.


```solidity

154   function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154

```solidity

198   function setRewardTokenSpeeds(
        VToken[] memory vTokens,
        uint256[] memory supplySpeeds,
        uint256[] memory borrowSpeeds
    ) external {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L198-L201


## [G-09]Use double if statements instead of &&


```solidity
755   if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
```    
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755

```solidity
File: /main/contracts/Lens/PoolLens.sol
506    if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506


```solidity
526    if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526

```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
261    if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261

```solidity

348     if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L348

```solidity

386    if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L386

```solidity
418    if (amount > 0 && amount <= rewardTokenRemaining) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L418

```solidity
File: /main/contracts/VToken.sol
837   if (redeemTokens == 0 && redeemAmount > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837


## [G-10] Can Make The Variable Outside The Loop To Save Gas 

```solidity

615   uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L615


```solidity
933   address rewardToken = address(rewardsDistributors[i].rewardToken());
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L933


```solidity
1123  address rewardToken = address(rewardsDistributors[i].rewardToken());
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1123

```solidity
File: /main/contracts/Lens/PoolLens.sol
431   uint256 borrowReward = calculateBorrowerReward(
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L431

```solidity
438   uint256 supplyReward = calculateSupplierReward(
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L438

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
356   address comptroller = _poolsByID[i];
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L356

```solidity
File: /main/contracts/RiskFund/RiskFund.sol

168   address comptroller = address(vToken.comptroller());
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L168

```solidity
174   uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L174

```solidity
File: /main/contracts/Shortfall/Shortfall.sol
390   uint256 marketBadDebt = vTokens[i].badDebt();
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L390

```solidity
393    uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L393



## [G-11] Make 3 event parameters indexed when possible
```solidity
File: /main/contracts/BaseJumpRateModelV2.sol
45   event NewInterestParams(
        uint256 baseRatePerBlock,
        uint256 multiplierPerBlock,
        uint256 jumpMultiplierPerBlock,
        uint256 kink
    );
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L45

```solidity
File: /main/contracts/MaxLoopsLimitHelper.sol
16   event MaxLoopsLimitUpdated(uint256 oldMaxLoopsLimit, uint256 newmaxLoopsLimit);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L16

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
118   event PoolNameSet(address indexed comptroller, string oldName, string newName);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L118

```solidity
123   event PoolMetadataUpdated(
        address indexed comptroller,
        VenusPoolMetaData oldMetadata,
        VenusPoolMetaData newMetadata
    );

132  event MarketAdded(address indexed comptroller, address vTokenAddress);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L118

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
22      event FundsReleased(address comptroller, address asset, uint256 amount);


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L22

```solidity
File: /main/contracts/RiskFund/ReserveHelpers.sol
30       event AssetsReservesUpdated(address indexed comptroller, address indexed asset, uint256 amount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L30


```solidity
57    event DistributedSupplierRewardToken(
        VToken indexed vToken,
        address indexed supplier,
        uint256 rewardTokenDelta,
        uint256 rewardTokenTotal,
        uint256 rewardTokenSupplyIndex
    );

75   event RewardTokenSupplySpeedUpdated(VToken indexed vToken, uint256 newSpeed);

78       event RewardTokenBorrowSpeedUpdated(VToken indexed vToken, uint256 newSpeed);

81       event RewardTokenGranted(address recipient, uint256 amount);

84   event ContributorRewardTokenSpeedUpdated(address indexed contributor, uint256 newSpeed);

87   event MarketInitialized(address vToken);

90   event RewardTokenSupplyIndexUpdated(address vToken);

93   event RewardTokenBorrowIndexUpdated(address vToken, Exp marketBorrowIndex);

96   event ContributorRewardsUpdated(address contributor, uint256 rewardAccrued);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L57

```solidity

47    event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);

    
50    event MinAmountToConvertUpdated(uint256 oldMinAmountToConvert, uint256 newMinAmountToConvert);

    
    event TransferredReserveForAuction(address comptroller, uint256 amount);

56   event TransferredReserveForAuction(address comptroller, uint256 amount);    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L47


```solidity
75    event AuctionStarted(
        address indexed comptroller,
        uint256 auctionStartBlock,
        AuctionType auctionType,
        VToken[] markets,
        uint256[] marketsDebt,
        uint256 seizedRiskFund,
        uint256 startBidBps
    );

86   event BidPlaced(address indexed comptroller, uint256 auctionStartBlock, uint256 bidBps, address indexed bidder);

89    event AuctionClosed(
        address indexed comptroller,
        uint256 auctionStartBlock,
        address indexed highestBidder,
        uint256 highestBidBps,
        uint256 seizedRiskFind,
        VToken[] markets,
        uint256[] marketDebt
    );
100   event AuctionRestarted(address indexed comptroller, uint256 auctionStartBlock);

106   event MinimumPoolBadDebtUpdated(uint256 oldMinimumPoolBadDebt, uint256 newMinimumPoolBadDebt);

109   event WaitForFirstBidderUpdated(uint256 oldWaitForFirstBidder, uint256 newWaitForFirstBidder);

112   event NextBidderBlockLimitUpdated(uint256 oldNextBidderBlockLimit, uint256 newNextBidderBlockLimit);

115   event IncentiveBpsUpdated(uint256 oldIncentiveBps, uint256 newIncentiveBps);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L75

```solidity
File: /main/contracts/VTokenInterfaces.sol
149       event AccrueInterest(uint256 cashPrior, uint256 interestAccumulated, uint256 borrowIndex, uint256 totalBorrows);

154        event Mint(address indexed minter, uint256 mintAmount, uint256 mintTokens, uint256 accountBalance);

159       event Redeem(address indexed redeemer, uint256 redeemAmount, uint256 redeemTokens, uint256 accountBalance);

164       event Borrow(address indexed borrower, uint256 borrowAmount, uint256 accountBorrows, uint256 totalBorrows);
169    event RepayBorrow(
        address indexed payer,
        address indexed borrower,
        uint256 repayAmount,
        uint256 accountBorrows,
        uint256 totalBorrows
    );
184   event BadDebtIncreased(address indexed borrower, uint256 badDebtDelta, uint256 badDebtOld, uint256 badDebtNew);

191   event BadDebtRecovered(uint256 badDebtOld, uint256 badDebtNew);

232  event NewProtocolSeizeShare(uint256 oldProtocolSeizeShareMantissa, uint256 newProtocolSeizeShareMantissa);

237   event NewReserveFactor(uint256 oldReserveFactorMantissa, uint256 newReserveFactorMantissa);

242  event ReservesAdded(address indexed benefactor, uint256 addAmount, uint256 newTotalReserves);

247   event ReservesReduced(address indexed admin, uint256 reduceAmount, uint256 newTotalReserves);

252  event Transfer(address indexed from, address indexed to, uint256 amount);

262  event HealBorrow(address payer, address borrower, uint256 repayAmount);

267   event SweepToken(address token);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L149

```solidity
File: /main/contracts/Factories/VTokenProxyFactory.sol
26   event VTokenProxyDeployed(VTokenArgs args);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L26

```solidity
File: /main/contracts/WhitePaperInterestRateModel.sol
29   event NewInterestParams(uint256 baseRatePerBlock, uint256 multiplierPerBlock);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L29


## [G-12] Use hardcode address instead address(this)

```solidity
File: /main/contracts/BaseJumpRateModelV2.sol
98    revert Unauthorized(msg.sender, address(this), signature);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L98

```solidity
File: /main/contracts/Comptroller.sol
501    if (seizerContract == address(this)) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#501

```solidity
504    if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#504


```solidity
File: /main/contracts/VToken.sol
420    emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L420
```solidity
527     uint256 balance = token.balanceOf(address(this));
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L527
```solidity
749     comptroller.preMintHook(address(this), minter, mintAmount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L749
```solidity
842     comptroller.preRedeemHook(address(this), redeemer, redeemTokens);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L842
```solidity
869     emit Transfer(redeemer, address(this), redeemTokens);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L869
```solidity
879     comptroller.preBorrowHook(address(this), borrower, borrowAmount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L879
```solidity
936     comptroller.preRepayHook(address(this), borrower);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L936
```solidity
1078    if (address(vTokenCollateral) == address(this)) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1078
```solidity
1079   _seize(address(this), liquidator, borrower, seizeTokens);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1079
```silidity
1104    comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1104
```solidity
1134   emit Transfer(borrower, address(this), protocolSeizeTokens);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1134
```solidity
1135  emit ReservesAdded(address(this), protocolSeizeAmount, totalReservesNew);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1135


## [G-13] Sort Solidity operations using short-circuit mode

```solidity
File: /main/contracts/Comptroller.sol
449   if (skipLiquidityCheck || isDeprecated(VToken(vTokenBorrowed))) {  
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L449