# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 367 at contests/venusContest/contracts/Comptroller.sol:

        if (snapshot.shortfall > 0) {


Found in line 1262 at contests/venusContest/contracts/Comptroller.sol:

        if (snapshot.shortfall > 0) {


Found in line 817 at contests/venusContest/contracts/VToken.sol:

        /* If redeemTokensIn > 0: */


Found in line 818 at contests/venusContest/contracts/VToken.sol:

        if (redeemTokensIn > 0) {


Found in line 837 at contests/venusContest/contracts/VToken.sol:

        if (redeemTokens == 0 && redeemAmount > 0) {


Found in line 1369 at contests/venusContest/contracts/VToken.sol:

        require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");


Found in line 82 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");


Found in line 83 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");


Found in line 139 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");


Found in line 244 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        if (balanceOfUnderlyingAsset > 0) {


Found in line 261 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        if (deltaBlocks > 0 && rewardTokenSpeed > 0) {


Found in line 418 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        if (amount > 0 && amount <= rewardTokenRemaining) {


Found in line 435 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        if (deltaBlocks > 0 && supplySpeed > 0) {


Found in line 438 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

            Double memory ratio = supplyTokens > 0


Found in line 446 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        } else if (deltaBlocks > 0) {


Found in line 463 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        if (deltaBlocks > 0 && borrowSpeed > 0) {


Found in line 466 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

            Double memory ratio = borrowAmount > 0


Found in line 474 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        } else if (deltaBlocks > 0) {


Found in line 462 at contests/venusContest/contracts/Lens/PoolLens.sol:

        if (deltaBlocks > 0 && borrowSpeed > 0) {


Found in line 466 at contests/venusContest/contracts/Lens/PoolLens.sol:

            Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });


Found in line 470 at contests/venusContest/contracts/Lens/PoolLens.sol:

        } else if (deltaBlocks > 0) {


Found in line 483 at contests/venusContest/contracts/Lens/PoolLens.sol:

        if (deltaBlocks > 0 && supplySpeed > 0) {


Found in line 486 at contests/venusContest/contracts/Lens/PoolLens.sol:

            Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });


Found in line 490 at contests/venusContest/contracts/Lens/PoolLens.sol:

        } else if (deltaBlocks > 0) {


Found in line 506 at contests/venusContest/contracts/Lens/PoolLens.sol:

        if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {


Found in line 526 at contests/venusContest/contracts/Lens/PoolLens.sol:

        if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 22 at contests/venusContest/contracts/ExponentialNoError.sol:

    uint256 internal constant halfExpScale = expScale / 2;

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 399 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        _numberOfPools++;

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 72 at contests/venusContest/contracts/BaseJumpRateModelV2.sol:

        require(address(accessControlManager_) != address(0), "invalid ACM address");


Found in line 26 at contests/venusContest/contracts/MaxLoopsLimitHelper.sol:

        require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");


Found in line 128 at contests/venusContest/contracts/Comptroller.sol:

        require(poolRegistry_ != address(0), "invalid pool registry address");


Found in line 692 at contests/venusContest/contracts/Comptroller.sol:

            require(borrowBalance == 0, "Nonzero borrow balance after liquidation");


Found in line 704 at contests/venusContest/contracts/Comptroller.sol:

        require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");


Found in line 705 at contests/venusContest/contracts/Comptroller.sol:

        require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");


Found in line 780 at contests/venusContest/contracts/Comptroller.sol:

        require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");


Found in line 808 at contests/venusContest/contracts/Comptroller.sol:

        require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure its really a VToken


Found in line 842 at contests/venusContest/contracts/Comptroller.sol:

        require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");


Found in line 866 at contests/venusContest/contracts/Comptroller.sol:

        require(vTokensCount != 0, "invalid number of markets");


Found in line 867 at contests/venusContest/contracts/Comptroller.sol:

        require(vTokensCount == newSupplyCaps.length, "invalid number of markets");


Found in line 928 at contests/venusContest/contracts/Comptroller.sol:

        require(!rewardsDistributorExists[address(_rewardsDistributor)], "already exists");


Found in line 962 at contests/venusContest/contracts/Comptroller.sol:

        require(address(newOracle) != address(0), "invalid price oracle address");


Found in line 1229 at contests/venusContest/contracts/Comptroller.sol:

        require(markets[market].isListed, "cannot pause a market that is not listed");


Found in line 34 at contests/venusContest/contracts/VToken.sol:

        require(_notEntered, "re-entered");


Found in line 72 at contests/venusContest/contracts/VToken.sol:

        require(admin_ != address(0), "invalid admin address");


Found in line 134 at contests/venusContest/contracts/VToken.sol:

        require(spender != address(0), "invalid spender address");


Found in line 196 at contests/venusContest/contracts/VToken.sol:

        require(minter != address(0), "invalid minter address");


Found in line 489 at contests/venusContest/contracts/VToken.sol:

        require(msg.sender == shortfall, "only shortfall contract can update bad debt");


Found in line 490 at contests/venusContest/contracts/VToken.sol:

        require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");


Found in line 525 at contests/venusContest/contracts/VToken.sol:

        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");


Found in line 526 at contests/venusContest/contracts/VToken.sol:

        require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");


Found in line 626 at contests/venusContest/contracts/VToken.sol:

        require(spender != address(0), "invalid spender address");


Found in line 646 at contests/venusContest/contracts/VToken.sol:

        require(spender != address(0), "invalid spender address");


Found in line 650 at contests/venusContest/contracts/VToken.sol:

        require(currentAllowance >= subtractedValue, "decreased allowance below zero");


Found in line 696 at contests/venusContest/contracts/VToken.sol:

        require(borrowRateMantissa <= borrowRateMaxMantissa, "borrow rate is absurdly high");


Found in line 805 at contests/venusContest/contracts/VToken.sol:

        require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");


Found in line 838 at contests/venusContest/contracts/VToken.sol:

            revert("redeemTokens zero");


Found in line 1072 at contests/venusContest/contracts/VToken.sol:

        require(amountSeizeError == NO_ERROR, "LIQUIDATE_COMPTROLLER_CALCULATE_AMOUNT_SEIZE_FAILED");


Found in line 1075 at contests/venusContest/contracts/VToken.sol:

        require(vTokenCollateral.balanceOf(borrower) >= seizeTokens, "LIQUIDATE_SEIZE_TOO_MUCH");


Found in line 1141 at contests/venusContest/contracts/VToken.sol:

        require(newComptroller.isComptroller(), "marker method returned false");


Found in line 1256 at contests/venusContest/contracts/VToken.sol:

        require(newInterestRateModel.isInterestRateModel(), "marker method returned false");


Found in line 1365 at contests/venusContest/contracts/VToken.sol:

        require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");


Found in line 1369 at contests/venusContest/contracts/VToken.sol:

        require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");


Found in line 8 at contests/venusContest/contracts/Proxy/UpgradeableBeacon.sol:

        require(implementation_ != address(0), "Invalid implementation address");


Found in line 137 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(convertibleBaseAsset_ != address(0), "invalid base asset address");


Found in line 138 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(address(riskFund_) != address(0), "invalid risk fund address");


Found in line 139 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(minimumPoolBadDebt_ != 0, "invalid minimum pool bad debt");


Found in line 161 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_isStarted(auction), "no on-going auction");


Found in line 162 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(!_isStale(auction), "auction is stale, restart it");


Found in line 163 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");


Found in line 212 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_isStarted(auction), "no on-going auction");


Found in line 278 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_isStarted(auction), "no on-going auction");


Found in line 279 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_isStale(auction), "you need to wait for more time for first bidder");


Found in line 295 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_nextBidderBlockLimit != 0, "_nextBidderBlockLimit must not be 0");


Found in line 309 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_incentiveBps != 0, "incentiveBps must not be 0");


Found in line 349 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(_poolRegistry != address(0), "invalid address");


Found in line 361 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");


Found in line 401 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        require(poolBadDebt >= minimumPoolBadDebt, "pool bad debt is too low");


Found in line 39 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");


Found in line 40 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        require(asset != address(0), "ReserveHelpers: Asset address invalid");


Found in line 51 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");


Found in line 52 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        require(asset != address(0), "ReserveHelpers: Asset address invalid");


Found in line 53 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");


Found in line 80 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");


Found in line 81 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");


Found in line 82 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");


Found in line 83 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");


Found in line 100 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");


Found in line 111 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");


Found in line 127 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");


Found in line 139 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");


Found in line 157 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");


Found in line 158 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");


Found in line 159 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");


Found in line 171 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

            require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");


Found in line 172 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

            require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");


Found in line 191 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");


Found in line 192 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");


Found in line 231 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");


Found in line 232 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

        require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");


Found in line 253 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

                    require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");


Found in line 40 at contests/venusContest/contracts/RiskFund/ProtocolShareReserve.sol:

        require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");


Found in line 41 at contests/venusContest/contracts/RiskFund/ProtocolShareReserve.sol:

        require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");


Found in line 54 at contests/venusContest/contracts/RiskFund/ProtocolShareReserve.sol:

        require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");


Found in line 71 at contests/venusContest/contracts/RiskFund/ProtocolShareReserve.sol:

        require(asset != address(0), "ProtocolShareReserve: Asset address invalid");


Found in line 72 at contests/venusContest/contracts/RiskFund/ProtocolShareReserve.sol:

        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");


Found in line 99 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        require(address(comptroller) == msg.sender, "Only comptroller can call this function");


Found in line 183 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        require(amountLeft == 0, "insufficient rewardToken for grant");


Found in line 284 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

            require(comptroller.isMarketListed(vToken), "market must be listed");


Found in line 309 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        require(comptroller.isMarketListed(vToken), "rewardToken market is not listed");


Found in line 225 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");


Found in line 226 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");


Found in line 257 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");


Found in line 258 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(input.asset != address(0), "PoolRegistry: Invalid asset address");


Found in line 259 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");


Found in line 260 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");


Found in line 396 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");


Found in line 440 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(bytes(name).length <= 100, "Pool's name is too large");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 5 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 154 at contests/venusContest/contracts/Comptroller.sol:

    function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {


Found in line 29 at contests/venusContest/contracts/ExponentialNoError.sol:

    function truncate(Exp memory exp) internal pure returns (uint256) {


Found in line 38 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {


Found in line 59 at contests/venusContest/contracts/ExponentialNoError.sol:

    function lessThanExp(Exp memory left, Exp memory right) internal pure returns (bool) {


Found in line 63 at contests/venusContest/contracts/ExponentialNoError.sol:

    function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {


Found in line 68 at contests/venusContest/contracts/ExponentialNoError.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 73 at contests/venusContest/contracts/ExponentialNoError.sol:

    function add_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {


Found in line 77 at contests/venusContest/contracts/ExponentialNoError.sol:

    function add_(Double memory a, Double memory b) internal pure returns (Double memory) {


Found in line 85 at contests/venusContest/contracts/ExponentialNoError.sol:

    function sub_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {


Found in line 89 at contests/venusContest/contracts/ExponentialNoError.sol:

    function sub_(Double memory a, Double memory b) internal pure returns (Double memory) {


Found in line 97 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {


Found in line 101 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(Exp memory a, uint256 b) internal pure returns (Exp memory) {


Found in line 105 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(uint256 a, Exp memory b) internal pure returns (uint256) {


Found in line 109 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(Double memory a, Double memory b) internal pure returns (Double memory) {


Found in line 113 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(Double memory a, uint256 b) internal pure returns (Double memory) {


Found in line 117 at contests/venusContest/contracts/ExponentialNoError.sol:

    function mul_(uint256 a, Double memory b) internal pure returns (uint256) {


Found in line 125 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(Exp memory a, Exp memory b) internal pure returns (Exp memory) {


Found in line 129 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(Exp memory a, uint256 b) internal pure returns (Exp memory) {


Found in line 133 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(uint256 a, Exp memory b) internal pure returns (uint256) {


Found in line 137 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(Double memory a, Double memory b) internal pure returns (Double memory) {


Found in line 141 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(Double memory a, uint256 b) internal pure returns (Double memory) {


Found in line 145 at contests/venusContest/contracts/ExponentialNoError.sol:

    function div_(uint256 a, Double memory b) internal pure returns (uint256) {


Found in line 28 at contests/venusContest/contracts/Factories/VTokenProxyFactory.sol:

    function deployVTokenProxy(VTokenArgs memory input) external returns (VToken) {


Found in line 187 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

    function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {


Found in line 277 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

    function claimRewardToken(address holder, VToken[] memory vTokens) public {


Found in line 458 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

    function _updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) internal {


Found in line 309 at contests/venusContest/contracts/Lens/PoolLens.sol:

    function getPoolDataFromVenusPool(address poolRegistryAddress, PoolRegistry.VenusPool memory venusPool)


Found in line 388 at contests/venusContest/contracts/Lens/PoolLens.sol:

    function vTokenMetadataAll(VToken[] memory vTokens) public view returns (VTokenMetadata[] memory) {


Found in line 255 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

    function addMarket(AddMarketInput memory input) external {


Found in line 343 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

    function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 6 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 229 at contests/venusContest/contracts/Lens/PoolLens.sol:

        for (uint256 i; i < rewardsDistributors.length; ++i) {


Found in line 263 at contests/venusContest/contracts/Lens/PoolLens.sol:

        for (uint256 i; i < markets.length; ++i) {


Found in line 418 at contests/venusContest/contracts/Lens/PoolLens.sol:

        for (uint256 i; i < markets.length; ++i) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 7 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 1399 at contests/venusContest/contracts/VToken.sol:

        if (shortfall_ == address(0)) {


Found in line 1408 at contests/venusContest/contracts/VToken.sol:

        if (protocolShareReserve_ == address(0)) {


Found in line 167 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||


Found in line 170 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),


Found in line 468 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        bool noBidder = auction.highestBidder == address(0);


Found in line 264 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

            _vTokens[input.comptroller][input.asset] == address(0),


Found in line 396 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");


Found in line 422 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        if (address(shortfall_) == address(0)) {


Found in line 431 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        if (protocolShareReserve_ == address(0)) {

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 8 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 45 at contests/venusContest/contracts/VTokenInterfaces.sol:

    uint8 public decimals;


Found in line 66 at contests/venusContest/contracts/VToken.sol:

        uint8 decimals_,


Found in line 1357 at contests/venusContest/contracts/VToken.sol:

        uint8 decimals_,


Found in line 68 at contests/venusContest/contracts/ExponentialNoError.sol:

    function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {


Found in line 70 at contests/venusContest/contracts/ExponentialNoError.sol:

        return uint32(n);


Found in line 18 at contests/venusContest/contracts/Factories/VTokenProxyFactory.sol:

        uint8 decimals_;


Found in line 19 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        uint32 block;


Found in line 126 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");


Found in line 433 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");


Found in line 461 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");


Found in line 99 at contests/venusContest/contracts/Lens/PoolLens.sol:

        uint32 block;


Found in line 36 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        uint8 decimals;

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 9 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 665 at contests/venusContest/contracts/VToken.sol:

    function exchangeRateCurrent() public override nonReentrant returns (uint256) {


Found in line 678 at contests/venusContest/contracts/VToken.sol:

    function accrueInterest() public virtual override returns (uint256) {


Found in line 214 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

    function updateAssetsState(address comptroller, address asset) public override(IRiskFund, ReserveHelpers) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 10 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 630 at contests/venusContest/contracts/VToken.sol:

        newAllowance += addedValue;


Found in line 66 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

            assetsReserves[asset] += balanceDifference;


Found in line 67 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

            poolsAssetsReserves[comptroller][asset] += balanceDifference;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 11 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 98 at contests/venusContest/contracts/BaseJumpRateModelV2.sol:

            revert Unauthorized(msg.sender, address(this), signature);


Found in line 501 at contests/venusContest/contracts/Comptroller.sol:

        if (seizerContract == address(this)) {


Found in line 504 at contests/venusContest/contracts/Comptroller.sol:

            if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {


Found in line 420 at contests/venusContest/contracts/VToken.sol:

            emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);


Found in line 527 at contests/venusContest/contracts/VToken.sol:

        uint256 balance = token.balanceOf(address(this));


Found in line 749 at contests/venusContest/contracts/VToken.sol:

        comptroller.preMintHook(address(this), minter, mintAmount);


Found in line 842 at contests/venusContest/contracts/VToken.sol:

        comptroller.preRedeemHook(address(this), redeemer, redeemTokens);


Found in line 869 at contests/venusContest/contracts/VToken.sol:

        emit Transfer(redeemer, address(this), redeemTokens);


Found in line 879 at contests/venusContest/contracts/VToken.sol:

        comptroller.preBorrowHook(address(this), borrower, borrowAmount);


Found in line 936 at contests/venusContest/contracts/VToken.sol:

        comptroller.preRepayHook(address(this), borrower);


Found in line 1027 at contests/venusContest/contracts/VToken.sol:

            address(this),


Found in line 1068 at contests/venusContest/contracts/VToken.sol:

            address(this),


Found in line 1078 at contests/venusContest/contracts/VToken.sol:

        if (address(vTokenCollateral) == address(this)) {


Found in line 1079 at contests/venusContest/contracts/VToken.sol:

            _seize(address(this), liquidator, borrower, seizeTokens);


Found in line 1104 at contests/venusContest/contracts/VToken.sol:

        comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);


Found in line 1134 at contests/venusContest/contracts/VToken.sol:

        emit Transfer(borrower, address(this), protocolSeizeTokens);


Found in line 1135 at contests/venusContest/contracts/VToken.sol:

        emit ReservesAdded(address(this), protocolSeizeAmount, totalReservesNew);


Found in line 1274 at contests/venusContest/contracts/VToken.sol:

        uint256 balanceBefore = token.balanceOf(address(this));


Found in line 1275 at contests/venusContest/contracts/VToken.sol:

        token.safeTransferFrom(from, address(this), amount);


Found in line 1276 at contests/venusContest/contracts/VToken.sol:

        uint256 balanceAfter = token.balanceOf(address(this));


Found in line 1304 at contests/venusContest/contracts/VToken.sol:

        comptroller.preTransferHook(address(this), src, dst, tokens);


Found in line 1423 at contests/venusContest/contracts/VToken.sol:

        return token.balanceOf(address(this));


Found in line 187 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);


Found in line 193 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);


Found in line 59 at contests/venusContest/contracts/RiskFund/ReserveHelpers.sol:

        uint256 currentBalance = IERC20Upgradeable(asset).balanceOf(address(this));


Found in line 264 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

                        address(this),


Found in line 417 at contests/venusContest/contracts/Rewards/RewardsDistributor.sol:

        uint256 rewardTokenRemaining = rewardToken.balanceOf(address(this));


Found in line 415 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        uint256 balanceBefore = token.balanceOf(address(this));


Found in line 416 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        token.safeTransferFrom(from, address(this), amount);


Found in line 417 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        uint256 balanceAfter = token.balanceOf(address(this));

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 12 

## [GAS] Do not calculate constants
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 106 at contests/venusContest/contracts/ComptrollerStorage.sol:

    uint256 internal constant closeFactorMinMantissa = 0.05e18; // 0.05


Found in line 109 at contests/venusContest/contracts/ComptrollerStorage.sol:

    uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9


Found in line 112 at contests/venusContest/contracts/ComptrollerStorage.sol:

    uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9


Found in line 22 at contests/venusContest/contracts/ExponentialNoError.sol:

    uint256 internal constant halfExpScale = expScale / 2;

------------------------------------------------------------------------ 

### Mitigation 

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.










# VULN 13 

## [GAS] Using private rather than public for constants, saves gas
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 22 at contests/venusContest/contracts/WhitePaperInterestRateModel.sol:

    uint256 public immutable multiplierPerBlock;


Found in line 27 at contests/venusContest/contracts/WhitePaperInterestRateModel.sol:

    uint256 public immutable baseRatePerBlock;

------------------------------------------------------------------------ 

### Mitigation 

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.










# VULN 14 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 60 at contests/venusContest/contracts/ExponentialNoError.sol:

        return left.mantissa < right.mantissa;


Found in line 469 at contests/venusContest/contracts/Shortfall/Shortfall.sol:

        return noBidder && (block.number > auction.startBlock + waitForFirstBidder);

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 15 

## [GAS] Use double require instead of using &&
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 842 at contests/venusContest/contracts/Comptroller.sol:

        require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");



Found in line 1365 at contests/venusContest/contracts/VToken.sol:

        require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");


------------------------------------------------------------------------ 

### Mitigation 

Using double require instead of operator && can save more gas. When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.
