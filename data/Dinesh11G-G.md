## 1st BUG

## Title
Cache Array Length Outside of Loop

## Links to affected code
  2023-05-venus/contracts/Comptroller.sol::155 => uint256 len = vTokens.length;
  2023-05-venus/contracts/Comptroller.sol::157 => uint256 accountAssetsLen = accountAssets[msg.sender].length;
  2023-05-venus/contracts/Comptroller.sol::214 => uint256 len = userAssetList.length;
  2023-05-venus/contracts/Comptroller.sol::229 => storedList[assetIndex] = storedList[storedList.length - 1];
  2023-05-venus/contracts/Comptroller.sol::272 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::302 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::374 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::400 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::519 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::556 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::580 => uint256 userAssetsCount = userAssets.length;
  2023-05-venus/contracts/Comptroller.sol::665 => uint256 ordersCount = orders.length;
  2023-05-venus/contracts/Comptroller.sol::688 => uint256 marketsCount = borrowMarkets.length;
  2023-05-venus/contracts/Comptroller.sol::817 => uint256 rewardDistributorsCount = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::839 => uint256 numMarkets = vTokens.length;
  2023-05-venus/contracts/Comptroller.sol::840 => uint256 numBorrowCaps = newBorrowCaps.length;
  2023-05-venus/contracts/Comptroller.sol::864 => uint256 vTokensCount = vTokens.length;
  2023-05-venus/contracts/Comptroller.sol::867 => require(vTokensCount == newSupplyCaps.length, "invalid number of markets");
  2023-05-venus/contracts/Comptroller.sol::892 => uint256 marketsCount = marketsList.length;
  2023-05-venus/contracts/Comptroller.sol::893 => uint256 actionsCount = actionsList.length;
  2023-05-venus/contracts/Comptroller.sol::930 => uint256 rewardsDistributorsLength = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::940 => uint256 rewardsDistributorsLen = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::946 => uint256 marketsCount = allMarkets.length;
  2023-05-venus/contracts/Comptroller.sol::1120 => uint256 rewardsDistributorsLength = rewardsDistributors.length;
  2023-05-venus/contracts/Comptroller.sol::1206 => uint256 marketsCount = allMarkets.length;
  2023-05-venus/contracts/Comptroller.sol::1214 => marketsCount = allMarkets.length;
  2023-05-venus/contracts/Comptroller.sol::1305 => uint256 assetsCount = assets.length;
  2023-05-venus/contracts/Lens/PoolLens.sol::125 => uint256 vTokenCount = vTokens.length;
  2023-05-venus/contracts/Lens/PoolLens.sol::141 => uint256 poolLength = venusPools.length;
  2023-05-venus/contracts/Lens/PoolLens.sol::206 => uint256 vTokenCount = vTokens.length;
  2023-05-venus/contracts/Lens/PoolLens.sol::228 => RewardSummary[] memory rewardSummary = new RewardSummary[](rewardsDistributors.length);
  2023-05-venus/contracts/Lens/PoolLens.sol::229 => for (uint256 i; i < rewardsDistributors.length; ++i) {
  2023-05-venus/contracts/Lens/PoolLens.sol::256 => BadDebt[] memory badDebts = new BadDebt[](markets.length);
  2023-05-venus/contracts/Lens/PoolLens.sol::263 => for (uint256 i; i < markets.length; ++i) {
  2023-05-venus/contracts/Lens/PoolLens.sol::389 => uint256 vTokenCount = vTokens.length;
  2023-05-venus/contracts/Lens/PoolLens.sol::417 => PendingReward[] memory pendingRewards = new PendingReward[](markets.length);
  2023-05-venus/contracts/Lens/PoolLens.sol::418 => for (uint256 i; i < markets.length; ++i) {
  2023-05-venus/contracts/Pool/PoolRegistry.sol::440 => require(bytes(name).length <= 100, "Pool's name is too large");
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::203 => uint256 numTokens = vTokens.length;
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::205 => numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::278 => uint256 vTokensCount = vTokens.length;
  2023-05-venus/contracts/RiskFund/RiskFund.sol::158 => require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::159 => require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::162 => uint256 marketsCount = markets.length;
  2023-05-venus/contracts/RiskFund/RiskFund.sol::255 => path[path.length - 1] == convertibleBaseAsset,
  2023-05-venus/contracts/RiskFund/RiskFund.sol::267 => totalAmount = amounts[path.length - 1];
  2023-05-venus/contracts/Shortfall/Shortfall.sol::174 => uint256 marketsCount = auction.markets.length;
  2023-05-venus/contracts/Shortfall/Shortfall.sol::218 => uint256 marketsCount = auction.markets.length;
  2023-05-venus/contracts/Shortfall/Shortfall.sol::373 => uint256 marketsCount = auction.markets.length;
  2023-05-venus/contracts/Shortfall/Shortfall.sol::382 => marketsCount = vTokens.length;
## Impact
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

## Proof of Concept
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

## Tools Used
Manual

## Recommended Mitigation Steps
uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}




## 2nd BUG

## Title
Use != 0 instead of > 0 for Unsigned Integer Comparison

## Links to affected code
  2023-05-venus/contracts/Comptroller.sol::367 => if (snapshot.shortfall > 0) {
  2023-05-venus/contracts/Comptroller.sol::1262 => if (snapshot.shortfall > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::462 => if (deltaBlocks > 0 && borrowSpeed > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::466 => Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });
  2023-05-venus/contracts/Lens/PoolLens.sol::470 => } else if (deltaBlocks > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::483 => if (deltaBlocks > 0 && supplySpeed > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::486 => Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });
  2023-05-venus/contracts/Lens/PoolLens.sol::490 => } else if (deltaBlocks > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::506 => if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
  2023-05-venus/contracts/Lens/PoolLens.sol::526 => if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::261 => if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::418 => if (amount > 0 && amount <= rewardTokenRemaining) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::435 => if (deltaBlocks > 0 && supplySpeed > 0) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::438 => Double memory ratio = supplyTokens > 0
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::446 => } else if (deltaBlocks > 0) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::463 => if (deltaBlocks > 0 && borrowSpeed > 0) {
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::466 => Double memory ratio = borrowAmount > 0
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::474 => } else if (deltaBlocks > 0) {
  2023-05-venus/contracts/RiskFund/RiskFund.sol::82 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::83 => require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::139 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::244 => if (balanceOfUnderlyingAsset > 0) {
  2023-05-venus/contracts/VToken.sol::817 => /* If redeemTokensIn > 0: */
  2023-05-venus/contracts/VToken.sol::818 => if (redeemTokensIn > 0) {
  2023-05-venus/contracts/VToken.sol::837 => if (redeemTokens == 0 && redeemAmount > 0) {
  2023-05-venus/contracts/VToken.sol::1369 => require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");\

## Impact
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

## Proof of Concept
🤦 Bad:

// `a` being of type unsigned integer
require(a > 0, "!a > 0");
🚀 Good:

// `a` being of type unsigned integer
require(a != 0, "!a > 0");

## Tools Used
Manual

## Recommended Mitigation Steps
// `a` being of type unsigned integer
require(a != 0, "!a > 0");





## 3rd BUG

## Title
Long Revert Strings

## Links to affected code
  2023-05-venus/contracts/BaseJumpRateModelV2.sol::4 => import "@venusprotocol/governance-contracts/contracts/Governance/IAccessControlManagerV8.sol";
  2023-05-venus/contracts/BaseJumpRateModelV2.sol::94 => string memory signature = "updateJumpRateModel(uint256,uint256,uint256,uint256)";
  2023-05-venus/contracts/Comptroller.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/Comptroller.sol::7 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/Comptroller.sol::11 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
  2023-05-venus/contracts/Comptroller.sol::692 => require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
  2023-05-venus/contracts/Comptroller.sol::704 => require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");
  2023-05-venus/contracts/Comptroller.sol::705 => require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");
  2023-05-venus/contracts/Comptroller.sol::731 => _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");
  2023-05-venus/contracts/Comptroller.sol::780 => require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");
  2023-05-venus/contracts/Comptroller.sol::837 => _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");
  2023-05-venus/contracts/Comptroller.sol::863 => _checkAccessAllowed("setMarketSupplyCaps(address[],uint256[])");
  2023-05-venus/contracts/Comptroller.sol::890 => _checkAccessAllowed("setActionsPaused(address[],uint256[],bool)");
  2023-05-venus/contracts/Comptroller.sol::913 => _checkAccessAllowed("setMinLiquidatableCollateral(uint256)");
  2023-05-venus/contracts/Comptroller.sol::936 => "distributor already exists with this reward"
  2023-05-venus/contracts/Comptroller.sol::1229 => require(markets[market].isListed, "cannot pause a market that is not listed");
  2023-05-venus/contracts/ComptrollerInterface.sol::4 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/ComptrollerStorage.sol::4 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/Factories/VTokenProxyFactory.sol::4 => import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
  2023-05-venus/contracts/Factories/VTokenProxyFactory.sol::7 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
  2023-05-venus/contracts/Factories/WhitePaperInterestRateModelFactory.sol::4 => import "../WhitePaperInterestRateModel.sol";
  2023-05-venus/contracts/Lens/PoolLens.sol::4 => import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
  2023-05-venus/contracts/Lens/PoolLens.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/Lens/PoolLens.sol::9 => import "../Pool/PoolRegistryInterface.sol";
  2023-05-venus/contracts/MaxLoopsLimitHelper.sol::26 => require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::6 => import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::7 => import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::10 => import "../Factories/VTokenProxyFactory.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::11 => import "../Factories/JumpRateModelFactory.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::12 => import "../Factories/WhitePaperInterestRateModelFactory.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::13 => import "../WhitePaperInterestRateModel.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::16 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlManager.sol";
  2023-05-venus/contracts/Pool/PoolRegistry.sol::223 => _checkAccessAllowed("createRegistryPool(string,address,uint256,uint256,uint256,address,uint256,address)");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::225 => require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::226 => require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::257 => require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::258 => require(input.asset != address(0), "PoolRegistry: Invalid asset address");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::259 => require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::260 => require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::265 => "PoolRegistry: Market already added for asset comptroller combination"
  2023-05-venus/contracts/Pool/PoolRegistry.sol::344 => _checkAccessAllowed("updatePoolMetadata(address,VenusPoolMetaData)");
  2023-05-venus/contracts/Pool/PoolRegistry.sol::396 => require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");
  2023-05-venus/contracts/Proxy/UpgradeableBeacon.sol::4 => import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::10 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::99 => require(address(comptroller) == msg.sender, "Only comptroller can call this function");
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::183 => require(amountLeft == 0, "insufficient rewardToken for grant");
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::202 => _checkAccessAllowed("setRewardTokenSpeeds(address[],uint256[],uint256[])");
  2023-05-venus/contracts/Rewards/RewardsDistributor.sol::206 => "RewardsDistributor::setRewardTokenSpeeds invalid input"
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::40 => require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::41 => require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::54 => require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::71 => require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
  2023-05-venus/contracts/RiskFund/ProtocolShareReserve.sol::72 => require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::4 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::7 => import "../Pool/PoolRegistryInterface.sol";
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::39 => require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::40 => require(asset != address(0), "ReserveHelpers: Asset address invalid");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::51 => require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::52 => require(asset != address(0), "ReserveHelpers: Asset address invalid");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::53 => require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
  2023-05-venus/contracts/RiskFund/ReserveHelpers.sol::56 => "ReserveHelpers: The pool doesn't support the asset"
  2023-05-venus/contracts/RiskFund/RiskFund.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/RiskFund/RiskFund.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/RiskFund/RiskFund.sol::13 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
  2023-05-venus/contracts/RiskFund/RiskFund.sol::80 => require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::81 => require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::82 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::83 => require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::100 => require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::111 => require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::114 => "Risk Fund: Base asset doesn't match"
  2023-05-venus/contracts/RiskFund/RiskFund.sol::127 => require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::139 => require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::156 => _checkAccessAllowed("swapPoolsAssets(address[],uint256[],address[][])");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::157 => require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::158 => require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::159 => require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::171 => require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::191 => require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::192 => require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::231 => require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::232 => require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::253 => require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
  2023-05-venus/contracts/RiskFund/RiskFund.sol::256 => "RiskFund: finally path must be convertible base asset"
  2023-05-venus/contracts/Shortfall/Shortfall.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::6 => import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::7 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::14 => import "../Pool/PoolRegistryInterface.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::15 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
  2023-05-venus/contracts/Shortfall/Shortfall.sol::163 => require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::215 => "waiting for next bidder. cannot close auction"
  2023-05-venus/contracts/Shortfall/Shortfall.sol::279 => require(_isStale(auction), "you need to wait for more time for first bidder");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::294 => _checkAccessAllowed("updateNextBidderBlockLimit(uint256)");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::295 => require(_nextBidderBlockLimit != 0, "_nextBidderBlockLimit must not be 0");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::322 => _checkAccessAllowed("updateMinimumPoolBadDebt(uint256)");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::335 => _checkAccessAllowed("updateWaitForFirstBidder(uint256)");
  2023-05-venus/contracts/Shortfall/Shortfall.sol::361 => require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
  2023-05-venus/contracts/VToken.sol::4 => import "@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol";
  2023-05-venus/contracts/VToken.sol::5 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
  2023-05-venus/contracts/VToken.sol::12 => import "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
  2023-05-venus/contracts/VToken.sol::13 => import "./RiskFund/IProtocolShareReserve.sol";
  2023-05-venus/contracts/VToken.sol::489 => require(msg.sender == shortfall, "only shortfall contract can update bad debt");
  2023-05-venus/contracts/VToken.sol::490 => require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");
  2023-05-venus/contracts/VToken.sol::525 => require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
  2023-05-venus/contracts/VToken.sol::526 => require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
  2023-05-venus/contracts/VToken.sol::805 => require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");
  2023-05-venus/contracts/VToken.sol::1072 => require(amountSeizeError == NO_ERROR, "LIQUIDATE_COMPTROLLER_CALCULATE_AMOUNT_SEIZE_FAILED");
  2023-05-venus/contracts/VToken.sol::1365 => require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");
  2023-05-venus/contracts/VToken.sol::1369 => require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
  2023-05-venus/contracts/VTokenInterfaces.sol::4 => import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
  2023-05-venus/contracts/VTokenInterfaces.sol::5 => import "@venusprotocol/oracle/contracts/PriceOracle.sol";

## Impact
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

## Proof of Concept
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

## Tools Used
Manual

## Recommended Mitigation Steps
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}


## 4th BUG

## Title
Use Shift Right/Left instead of Division/Multiplication if possible

## Links to affected code
  2023-05-venus/contracts/ExponentialNoError.sol::22 => uint256 internal constant halfExpScale = expScale / 2;
  2023-05-venus/contracts/ExponentialNoError.sol::64 => require(n < 2**224, errorMessage);

## Impact
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

## Proof of Concept
🤦 Bad:

uint256 b = a / 2;
uint256 c = a / 4;
uint256 d = a * 8;
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;

## Tools Used
Manual

## Recommended Mitigation Steps
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;