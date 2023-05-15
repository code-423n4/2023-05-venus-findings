### Cache Array Length Outside of Loop

#### Findings:
```

contracts\Lens\PoolLens.sol::229 => for (uint256 i; i < rewardsDistributors.length; ++i) {
contracts\Lens\PoolLens.sol::263 => for (uint256 i; i < markets.length; ++i) {
contracts\Lens\PoolLens.sol::418 => for (uint256 i; i < markets.length; ++i) {
```

### Use != 0 instead of > 0 for Unsigned Integer Comparison


#### Findings:
```
contracts\Comptroller.sol::367 => if (snapshot.shortfall > 0) {
contracts\Comptroller.sol::1262 => if (snapshot.shortfall > 0) {
contracts\Lens\PoolLens.sol::462 => if (deltaBlocks > 0 && borrowSpeed > 0) {
contracts\Lens\PoolLens.sol::466 => Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });
contracts\Lens\PoolLens.sol::470 => } else if (deltaBlocks > 0) {
contracts\Lens\PoolLens.sol::483 => if (deltaBlocks > 0 && supplySpeed > 0) {
contracts\Lens\PoolLens.sol::486 => Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });
contracts\Lens\PoolLens.sol::490 => } else if (deltaBlocks > 0) {
contracts\Lens\PoolLens.sol::506 => if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
contracts\Lens\PoolLens.sol::526 => if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
contracts\Rewards\RewardsDistributor.sol::261 => if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
contracts\Rewards\RewardsDistributor.sol::418 => if (amount > 0 && amount <= rewardTokenRemaining) {
contracts\Rewards\RewardsDistributor.sol::435 => if (deltaBlocks > 0 && supplySpeed > 0) {
contracts\Rewards\RewardsDistributor.sol::438 => Double memory ratio = supplyTokens > 0
contracts\Rewards\RewardsDistributor.sol::446 => } else if (deltaBlocks > 0) {
contracts\Rewards\RewardsDistributor.sol::463 => if (deltaBlocks > 0 && borrowSpeed > 0) {
contracts\Rewards\RewardsDistributor.sol::466 => Double memory ratio = borrowAmount > 0
contracts\Rewards\RewardsDistributor.sol::474 => } else if (deltaBlocks > 0) {
contracts\RiskFund\RiskFund.sol::82 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
contracts\RiskFund\RiskFund.sol::83 => require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");
contracts\RiskFund\RiskFund.sol::139 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
contracts\RiskFund\RiskFund.sol::244 => if (balanceOfUnderlyingAsset > 0) {
contracts\VToken.sol::817 => /* If redeemTokensIn > 0: */
contracts\VToken.sol::818 => if (redeemTokensIn > 0) {
contracts\VToken.sol::837 => if (redeemTokens == 0 && redeemAmount > 0) {
contracts\VToken.sol::1369 => require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
```

### Long Revert Strings


#### Findings:
```
contracts\BaseJumpRateModelV2.sol::4 => import "@venusprotocol/governance-contracts/contracts/Governance/IAccessControlManagerV8.sol";
contracts\BaseJumpRateModelV2.sol::94 => string memory signature = "updateJumpRateModel(uint256,uint256,uint256,uint256)";
contracts\Comptroller.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\Comptroller.sol::7 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\Comptroller.sol::11 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
contracts\Comptroller.sol::692 => require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
contracts\Comptroller.sol::704 => require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");
contracts\Comptroller.sol::705 => require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");
contracts\Comptroller.sol::731 => _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");
contracts\Comptroller.sol::780 => require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");
contracts\Comptroller.sol::837 => _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");
contracts\Comptroller.sol::863 => _checkAccessAllowed("setMarketSupplyCaps(address[],uint256[])");
contracts\Comptroller.sol::890 => _checkAccessAllowed("setActionsPaused(address[],uint256[],bool)");
contracts\Comptroller.sol::913 => _checkAccessAllowed("setMinLiquidatableCollateral(uint256)");
contracts\Comptroller.sol::936 => "distributor already exists with this reward"
contracts\Comptroller.sol::1229 => require(markets[market].isListed, "cannot pause a market that is not listed");
contracts\ComptrollerInterface.sol::4 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\ComptrollerStorage.sol::4 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\Factories\VTokenProxyFactory.sol::4 => import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
contracts\Factories\VTokenProxyFactory.sol::7 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
contracts\Factories\WhitePaperInterestRateModelFactory.sol::4 => import "../WhitePaperInterestRateModel.sol";
contracts\Lens\PoolLens.sol::4 => import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
contracts\Lens\PoolLens.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\Lens\PoolLens.sol::9 => import "../Pool/PoolRegistryInterface.sol";
contracts\MaxLoopsLimitHelper.sol::26 => require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");
contracts\Pool\PoolRegistry.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\Pool\PoolRegistry.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\Pool\PoolRegistry.sol::6 => import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
contracts\Pool\PoolRegistry.sol::7 => import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
contracts\Pool\PoolRegistry.sol::10 => import "../Factories/VTokenProxyFactory.sol";
contracts\Pool\PoolRegistry.sol::11 => import "../Factories/JumpRateModelFactory.sol";
contracts\Pool\PoolRegistry.sol::12 => import "../Factories/WhitePaperInterestRateModelFactory.sol";
contracts\Pool\PoolRegistry.sol::13 => import "../WhitePaperInterestRateModel.sol";
contracts\Pool\PoolRegistry.sol::16 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
contracts\Pool\PoolRegistry.sol::223 => _checkAccessAllowed("createRegistryPool(string,address,uint256,uint256,uint256,address,uint256,address)");
contracts\Pool\PoolRegistry.sol::225 => require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");
contracts\Pool\PoolRegistry.sol::226 => require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");
contracts\Pool\PoolRegistry.sol::257 => require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");
contracts\Pool\PoolRegistry.sol::258 => require(input.asset != address(0), "PoolRegistry: Invalid asset address");
contracts\Pool\PoolRegistry.sol::259 => require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");
contracts\Pool\PoolRegistry.sol::260 => require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");
contracts\Pool\PoolRegistry.sol::265 => "PoolRegistry: Market already added for asset comptroller combination"
contracts\Pool\PoolRegistry.sol::344 => _checkAccessAllowed("updatePoolMetadata(address,VenusPoolMetaData)");
contracts\Pool\PoolRegistry.sol::396 => require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");
contracts\Proxy\UpgradeableBeacon.sol::4 => import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";
contracts\Rewards\RewardsDistributor.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\Rewards\RewardsDistributor.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\Rewards\RewardsDistributor.sol::10 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
contracts\Rewards\RewardsDistributor.sol::99 => require(address(comptroller) == msg.sender, "Only comptroller can call this function");
contracts\Rewards\RewardsDistributor.sol::183 => require(amountLeft == 0, "insufficient rewardToken for grant");
contracts\Rewards\RewardsDistributor.sol::202 => _checkAccessAllowed("setRewardTokenSpeeds(address[],uint256[],uint256[])");
contracts\Rewards\RewardsDistributor.sol::206 => "RewardsDistributor::setRewardTokenSpeeds invalid input"
contracts\RiskFund\ProtocolShareReserve.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\RiskFund\ProtocolShareReserve.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\RiskFund\ProtocolShareReserve.sol::40 => require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
contracts\RiskFund\ProtocolShareReserve.sol::41 => require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");
contracts\RiskFund\ProtocolShareReserve.sol::54 => require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");
contracts\RiskFund\ProtocolShareReserve.sol::71 => require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
contracts\RiskFund\ProtocolShareReserve.sol::72 => require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
contracts\RiskFund\ReserveHelpers.sol::4 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\RiskFund\ReserveHelpers.sol::7 => import "../Pool/PoolRegistryInterface.sol";
contracts\RiskFund\ReserveHelpers.sol::39 => require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
contracts\RiskFund\ReserveHelpers.sol::40 => require(asset != address(0), "ReserveHelpers: Asset address invalid");
contracts\RiskFund\ReserveHelpers.sol::51 => require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
contracts\RiskFund\ReserveHelpers.sol::52 => require(asset != address(0), "ReserveHelpers: Asset address invalid");
contracts\RiskFund\ReserveHelpers.sol::53 => require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
contracts\RiskFund\ReserveHelpers.sol::56 => "ReserveHelpers: The pool doesn't support the asset"
contracts\RiskFund\RiskFund.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\RiskFund\RiskFund.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\RiskFund\RiskFund.sol::13 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
contracts\RiskFund\RiskFund.sol::80 => require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
contracts\RiskFund\RiskFund.sol::81 => require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
contracts\RiskFund\RiskFund.sol::82 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
contracts\RiskFund\RiskFund.sol::83 => require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");
contracts\RiskFund\RiskFund.sol::100 => require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
contracts\RiskFund\RiskFund.sol::111 => require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");
contracts\RiskFund\RiskFund.sol::114 => "Risk Fund: Base asset doesn't match"
contracts\RiskFund\RiskFund.sol::127 => require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");
contracts\RiskFund\RiskFund.sol::139 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
contracts\RiskFund\RiskFund.sol::156 => _checkAccessAllowed("swapPoolsAssets(address[],uint256[],address[][])");
contracts\RiskFund\RiskFund.sol::157 => require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
contracts\RiskFund\RiskFund.sol::158 => require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
contracts\RiskFund\RiskFund.sol::159 => require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");
contracts\RiskFund\RiskFund.sol::171 => require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
contracts\RiskFund\RiskFund.sol::191 => require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");
contracts\RiskFund\RiskFund.sol::192 => require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");
contracts\RiskFund\RiskFund.sol::231 => require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");
contracts\RiskFund\RiskFund.sol::232 => require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");
contracts\RiskFund\RiskFund.sol::253 => require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
contracts\RiskFund\RiskFund.sol::256 => "RiskFund: finally path must be convertible base asset"
contracts\Shortfall\Shortfall.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\Shortfall\Shortfall.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\Shortfall\Shortfall.sol::6 => import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
contracts\Shortfall\Shortfall.sol::7 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
contracts\Shortfall\Shortfall.sol::14 => import "../Pool/PoolRegistryInterface.sol";
contracts\Shortfall\Shortfall.sol::15 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
contracts\Shortfall\Shortfall.sol::163 => require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
contracts\Shortfall\Shortfall.sol::215 => "waiting for next bidder. cannot close auction"
contracts\Shortfall\Shortfall.sol::279 => require(_isStale(auction), "you need to wait for more time for first bidder");
contracts\Shortfall\Shortfall.sol::294 => _checkAccessAllowed("updateNextBidderBlockLimit(uint256)");
contracts\Shortfall\Shortfall.sol::295 => require(_nextBidderBlockLimit != 0, "_nextBidderBlockLimit must not be 0");
contracts\Shortfall\Shortfall.sol::322 => _checkAccessAllowed("updateMinimumPoolBadDebt(uint256)");
contracts\Shortfall\Shortfall.sol::335 => _checkAccessAllowed("updateWaitForFirstBidder(uint256)");
contracts\Shortfall\Shortfall.sol::361 => require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
contracts\VToken.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
contracts\VToken.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
contracts\VToken.sol::12 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
contracts\VToken.sol::13 => import "./RiskFund/IProtocolShareReserve.sol";
contracts\VToken.sol::489 => require(msg.sender == shortfall, "only shortfall contract can update bad debt");
contracts\VToken.sol::490 => require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");
contracts\VToken.sol::525 => require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
contracts\VToken.sol::526 => require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
contracts\VToken.sol::805 => require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");
contracts\VToken.sol::1072 => require(amountSeizeError == NO_ERROR, "LIQUIDATE_COMPTROLLER_CALCULATE_AMOUNT_SEIZE_FAILED");
contracts\VToken.sol::1365 => require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");
contracts\VToken.sol::1369 => require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
contracts\VTokenInterfaces.sol::4 => import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
contracts\VTokenInterfaces.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
```

### Use Shift Right/Left instead of Division/Multiplication if possible


#### Findings:
```
contracts\ComptrollerStorage.sol::120 => * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
contracts\ExponentialNoError.sol::22 => uint256 internal constant halfExpScale = expScale / 2;
contracts\ExponentialNoError.sol::64 => require(n < 2**224, errorMessage);
contracts\MaxLoopsLimitHelper.sol::11 => * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
contracts\VTokenInterfaces.sol::128 => * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
```