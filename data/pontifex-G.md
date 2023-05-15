### GAS-1 Move `if`/validation statements as up as possible in the function body
There are 10 instances.
```solidity
333:        if (!markets[vToken].isListed) {

336:        if (!markets[vToken].accountMembership[borrower]) {

439:        if (!markets[vTokenBorrowed].isListed) {

442:        if (!markets[vTokenCollateral].isListed) {

750:        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {

941:        _ensureMaxLoops(rewardsDistributorsLen + 1);    
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L333
```solidity
1045:        if (borrower == liquidator) {

1050:        if (repayAmount == 0) {

1055:        if (repayAmount == type(uint256).max) {

1307:        if (src == dst) {    
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1045C1-L1057C10

### GAS-2 Emit can be rearranged
The emit can be rearranged in this way:
```solidity
707:        emit NewCloseFactor(closeFactorMantissa, newCloseFactorMantissa);
708:        closeFactorMantissa = newCloseFactorMantissa;
```
Instead of:
```solidity
707:        uint256 oldCloseFactorMantissa = closeFactorMantissa;
708:        closeFactorMantissa = newCloseFactorMantissa;
709:        emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL707C1-L709C74
With this arrangement, both the deploy time (~1.4 k * 4 = 5.6k gas per function) and every time the functions are triggered (~16 gas) gas savings will be achieved.
There are 20 other instances:
```solidity
791:        emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);

917:        emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);

966:        emit NewPriceOracle(oldOracle, newOracle);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L791
```solidity
 319:        emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);

1168:        emit NewReserveFactor(oldReserveFactorMantissa, newReserveFactorMantissa);

1262:        emit NewMarketInterestRateModel(oldInterestRateModel, newInterestRateModel);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1262C1-L1262C85
```solidity
298:        emit NextBidderBlockLimitUpdated(oldNextBidderBlockLimit, _nextBidderBlockLimit);

312:        emit IncentiveBpsUpdated(oldIncentiveBps, _incentiveBps);

325:        emit MinimumPoolBadDebtUpdated(oldMinimumPoolBadDebt, _minimumPoolBadDebt);

338:        emit WaitForFirstBidderUpdated(oldWaitForFirstBidder, _waitForFirstBidder);

352:        emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL298C1-L298C90
```solidity
337:        emit PoolNameSet(comptroller, oldName, name);

347:        emit PoolMetadataUpdated(comptroller, oldMetadata, _metadata);

427:        emit NewShortfallContract(oldShortfall, address(shortfall_));

436:        emit NewProtocolShareReserve(oldProtocolShareReserve, protocolShareReserve_);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL337C1-L337C54
```solidity
103:        emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);

119:        emit ShortfallContractUpdated(oldShortfallContractAddress, shortfallContractAddress_);

130:        emit PancakeSwapRouterUpdated(oldPancakeSwapRouter, pancakeSwapRouter_);

142:        emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL103C1-L103C66
```solidity
57:        emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ProtocolShareReserve.sol#L57

### GAS-3 Comptroller.sol_#L201_Redundant type casting
Use `vTokenAddress` instead.
```solidity
201:        Market storage marketToExit = markets[address(vToken)];
```

### GAS-4 Redundant caching variables
Use a state variable instead of a caching variable.
There are 4 instances:
Exlude `accountBorrowsPrev`:
```solidity
896:        uint256 accountBorrowsPrev = _borrowBalanceStored(borrower);
897:        uint256 accountBorrowsNew = accountBorrowsPrev + borrowAmount;
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L896-L897
Exlude `poolData`:
```solidity
147:            PoolData memory poolData = getPoolDataFromVenusPool(poolRegistryAddress, venusPool);
148:            poolDataItems[i] = poolData;
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL147C1-L148C41
Exlude `srcTokensNew` and `dstTokensNew`. Use `-=` and `+=` to change state variables:
```solidity
1321:        uint256 srcTokensNew = accountTokens[src] - tokens;
1322:        uint256 dstTokensNew = accountTokens[dst] + tokens;

1327:        accountTokens[src] = srcTokensNew;
1328:        accountTokens[dst] = dstTokensNew;
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1321C1-L1328C43


### GAS-5 Use existing caching variables instead of state variables or declaring new one
There are 6 instances:
Use existing `rewardsDistributorsLength` instead of `rewardsDistributorsLen`
```solidity
940:        uint256 rewardsDistributorsLen = rewardsDistributors.length;
941:        _ensureMaxLoops(rewardsDistributorsLen + 1);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL940C1-L941C53
Use existing `contributorAccrued` instead of `rewardTokenAccrued[contributor]`:
```solidity
268:            emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL265C1-L268C90
Declare `marketsCount` in the top and use instead of `markets.length`
```solidity
158:        require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
159:        require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");

162:        uint256 marketsCount = markets.length;
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL158C1-L162C47
Use existing `kink_` instead of `kink`:
```solidity
160:        kink = kink_;

162:        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L162
Declare `badDebtOld` in the top and use instead of `badDebt`
```solidity
490:        require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");

492:        uint256 badDebtOld = badDebt;
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL490C1-L492C38

### GAS-6 Use caching variables instead of state variables or multiple reading arrays values.
`marketsList[marketIdx]` can be cached before the loop:
```solidity
898:            for (uint256 actionIdx; actionIdx < actionsCount; ++actionIdx) {
899:                _setActionPaused(address(marketsList[marketIdx]), actionsList[actionIdx], paused);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL898C1-L899C99
`totalReserves` and `protocolShareReserve` should be cached:
```solidity
1215:        if (reduceAmount > totalReserves) {
1223:        totalReservesNew = totalReserves - reduceAmount;
1230:        _doTransferOut(protocolShareReserve, reduceAmount);
1233:        IProtocolShareReserve(protocolShareReserve).updateAssetsState(address(comptroller), underlying);
1235:        emit ReservesReduced(protocolShareReserve, reduceAmount, totalReservesNew);
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#LL1230C1-L1235C84