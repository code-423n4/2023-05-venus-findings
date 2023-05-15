### Gas Optimizations

Where the gas saved could be calculated, it is listed in the "Min Gas Savings (Est.)" column. It is not possible to calculate the savings for all instances since they depend on runtime conditions.

| Number | Issue | Instances | Min Gas Savings (Est.) |
| :-: | :-- | :-: | :-: |
| [G-01] | Use assembly to check for address(0) | 46 | 2,760 |
| [G-02] | Inequality comparisons | 26 | 78 |
| [G-03] | Addition and subtraction syntax | 6 | 678 |
| [G-04] | Check amount before transfer | 9 | indeterminate |
| [G-05] | Avoid storage variable assignment if possible | 23 | indeterminate |
| [G-06] | Remove unnecessary variables | 4 | indeterminate |

#### [G-01] Use assembly to check for address(0)

Use assembly to check for address(0). Example code is at [Solidity Assembly: Checking if an Address is 0 (Efficiently)](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331). Using the method described saves approximately 60 gas per call versus the Solidity `require` statement.

```solidity
File: contracts/BaseJumpRateModelV2.sol

72:    require(address(accessControlManager_) != address(0), "invalid ACM address");
```

```solidity
File: contracts/Comptroller.sol

128:    require(poolRegistry_ != address(0), "invalid pool registry address");

962:    require(address(newOracle) != address(0), "invalid price oracle address");
```

```solidity
File: contracts/VToken.sol

72:    require(admin_ != address(0), "invalid admin address");

134:   require(spender != address(0), "invalid spender address");

196:   require(minter != address(0), "invalid minter address");

626:   require(spender != address(0), "invalid spender address");

646:   require(spender != address(0), "invalid spender address");

1399:  if (shortfall_ == address(0)) {

1408:  if (protocolShareReserve_ == address(0)) {
```

```solidity
File: contracts/Pool/PoolRegistry.sol

225:    require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");
226:    require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

257:    require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");
258:    require(input.asset != address(0), "PoolRegistry: Invalid asset address");
259:    require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");
260:    require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

264:    _vTokens[input.comptroller][input.asset] == address(0),

396:    require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");

422:    if (address(shortfall_) == address(0)) {

431:    if (protocolShareReserve_ == address(0)) {
```

```solidity
File: contracts/Proxy/UpgradeableBeacon.sol

8:    require(implementation_ != address(0), "Invalid implementation address");
```

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

40:    require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
41:    require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

54:    require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

71:    require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
```

```solidity
File: contracts/RiskFund/ReserveHelpers.sol

40:    require(asset != address(0), "ReserveHelpers: Asset address invalid");

52:    require(asset != address(0), "ReserveHelpers: Asset address invalid");
53:    require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");

55:    PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
```

```solidity
File: contracts/RiskFund/RiskFund.sol

80:    require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
81:    require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

100:   require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

127:   require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

157:   require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
```

```solidity
File: contracts/Shortfall/Shortfall.sol

137:    require(convertibleBaseAsset_ != address(0), "invalid base asset address");
138:    require(address(riskFund_) != address(0), "invalid risk fund address");

166:    ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
167:        (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
168:    (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
169:        ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
170:            (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),

180:    if (auction.highestBidder != address(0)) {

189:    if (auction.highestBidder != address(0)) {

214:    block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),

349:    require(_poolRegistry != address(0), "invalid address");

468:    bool noBidder = auction.highestBidder == address(0);
```

#### [G-02] Inequality comparisons

The non-strict inequality comparisons `>=` or `<=` are cheaper than the strict inequality comparisons `>` and `<`. Strict inequalities add a check of non-equality which costs around 3 gas. Mitigation: Use `>=` and `<=` instead of `>` and `<` when possible and when doing so does not incur additional gas use (like if changing it requires some arithmetic to be performed).

```solidity
File: contracts/Comptroller.sol

1262:   if (snapshot.shortfall > 0) {
```

```solidity
File: contracts/VToken.sol

818:   if (redeemTokensIn > 0) {

837:   if (redeemTokens == 0 && redeemAmount > 0) {

1369:  require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
```

```solidity
File: contracts/Lens/PoolLens.sol

466:   Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });

470:   } else if (deltaBlocks > 0) {

483:   if (deltaBlocks > 0 && supplySpeed > 0) {

486:   Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });

490:   } else if (deltaBlocks > 0) {

506:   if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

526:   if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
```

```solidity
File: contracts/Rewards/RewardsDistributor.sol

261:   if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

418:   if (amount > 0 && amount <= rewardTokenRemaining) {

435:   if (deltaBlocks > 0 && supplySpeed > 0) {

438:   Double memory ratio = supplyTokens > 0

446:   } else if (deltaBlocks > 0) {

463:   if (deltaBlocks > 0 && borrowSpeed > 0) {

466:   Double memory ratio = borrowAmount > 0

474:   } else if (deltaBlocks > 0) {
```

```solidity
File: contracts/RiskFund/RiskFund.sol

82:    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
83:    require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

139:   require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

244:   if (balanceOfUnderlyingAsset > 0) {
```

#### [G-03] Addition and subtraction syntax

For state variables, `x = x - y` costs less gas than `x -= y`. Same for addition. Consider replacing all `-=` and `+=` occurrences to save gas. Using the addition/subtraction operator instead of plus/minus-equals [saves 113 gas](https://gist.github.com/ezcodeslide/0d106946dc4143ca8654048c1a9450f8).

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

74:    assetsReserves[asset] -= amount;
75:    poolsAssetsReserves[comptroller][asset] -= amount;
```

```solidity
File: contracts/RiskFund/RiskFund.sol

249:   assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
250:   poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
```

```solidity
File: contracts/RiskFund/ReserveHelpers.sol

66:    assetsReserves[asset] += balanceDifference;
67:    poolsAssetsReserves[comptroller][asset] += balanceDifference;
```

#### [G-04] Check amount before transfer

Verify an amount is greater than zero before making an external call to transfer it. If the amount is zero, the transfer can be avoided and the gas needed for an external call can be saved.

```solidity
File: contracts/VToken.sol

1286:    token.safeTransfer(to, amount);
```

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

81:    IERC20Upgradeable(asset).safeTransfer(protocolIncome, protocolIncomeAmount);
82:    IERC20Upgradeable(asset).safeTransfer(riskFund, amount - protocolIncomeAmount);
```

```solidity
File: contracts/RiskFund/RiskFund.sol

194:    IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);
```

```solidity
File: contracts/Shortfall/Shortfall.sol

183:    erc20.safeTransfer(auction.highestBidder, previousBidAmount);

190:    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);

229:    erc20.safeTransfer(address(auction.markets[i]), bidAmount);

232:    erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);

248:    IERC20Upgradeable(convertibleBaseAsset).safeTransfer(auction.highestBidder, riskFundBidAmount);
```

#### [G-05] Avoid storage variable assignment if possible

In Solidity, it costs ~20,000 gas more to write a storage variable than to read it. Writing to a storage variable results in a state change, which incurs gas costs for executing the transaction. On the other hand, reading a storage variable does not change the state, and it only requires a small amount of gas to access the variable value.

For “setter” functions that are exposed so that the value of a storage variable can be set, the new value should first be compared against the current value. If the new value is the same as the current value, the assignment of the storage variable, and oftentimes the emission of the event to notify of the change, can be skipped and therefore gas can be saved.

There is a good example of this best practice in Comptroller.sol:

```solidity
File: contracts/Comptroller.sol

759:    uint256 oldCollateralFactorMantissa = market.collateralFactorMantissa;
760:    if (newCollateralFactorMantissa != oldCollateralFactorMantissa) {
761:        market.collateralFactorMantissa = newCollateralFactorMantissa;
762:        emit NewCollateralFactor(vToken, oldCollateralFactorMantissa, newCollateralFactorMantissa);
763:    }
```

Instances that can be improved:

```solidity
File: contracts/Comptroller.sol

788:    liquidationIncentiveMantissa = newLiquidationIncentiveMantissa;
791:    emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);

847:    borrowCaps[address(vTokens[i])] = newBorrowCaps[i];
848:    emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);

872:    supplyCaps[address(vTokens[i])] = newSupplyCaps[i];
873:    emit NewSupplyCap(vTokens[i], newSupplyCaps[i]);

916:    minLiquidatableCollateral = newMinLiquidatableCollateral;
917:    emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);

965:    oracle = newOracle;
966:    emit NewPriceOracle(oldOracle, newOracle);

1230:   _actionPaused[market][action] = paused;
1231:   emit ActionPausedMarket(VToken(market), action, paused);
```

```solidity
File: contracts/VToken.sol

318:    protocolSeizeShareMantissa = newProtocolSeizeShareMantissa_;
319:    emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);

1144:    comptroller = newComptroller;
1147:    emit NewComptroller(oldComptroller, newComptroller);

1166:    reserveFactorMantissa = newReserveFactorMantissa;
1168:    emit NewReserveFactor(oldReserveFactorMantissa, newReserveFactorMantissa);

1259:    interestRateModel = newInterestRateModel;
1262:    emit NewMarketInterestRateModel(oldInterestRateModel, newInterestRateModel);

1403:    shortfall = shortfall_;
1404:    emit NewShortfallContract(oldShortfall, shortfall_);

1412:    protocolShareReserve = protocolShareReserve_;
1413:    emit NewProtocolShareReserve(oldProtocolShareReserve, address(protocolShareReserve_));
```

```solidity
File: contracts/Pool/PoolRegistry.sol

336:    _poolByComptroller[comptroller].name = name;
337:    emit PoolNameSet(comptroller, oldName, name);

426:    shortfall = shortfall_;
427:    emit NewShortfallContract(oldShortfall, address(shortfall_));

435:    protocolShareReserve = protocolShareReserve_;
436:    emit NewProtocolShareReserve(oldProtocolShareReserve, protocolShareReserve_);
```

```solidity
File: contracts/Rewards/RewardsDistributor.sol

318:    rewardTokenSupplySpeeds[address(vToken)] = supplySpeed;
319:    emit RewardTokenSupplySpeedUpdated(vToken, supplySpeed);

228:    rewardTokenContributorSpeeds[contributor] = rewardTokenSpeed;
230:    emit ContributorRewardTokenSpeedUpdated(contributor, rewardTokenSpeed);
```

```solidity
File: contracts/MaxLoopsLimitHelper.sol

29:    maxLoopsLimit = limit;
31:    emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
```

```solidity
File: contracts/RiskFund/ProtocolShareReserve.sol

56:    poolRegistry = _poolRegistry;
57:    emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
```

```solidity
File: contracts/RiskFund/RiskFund.sol

102:    poolRegistry = _poolRegistry;
103:    emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);

118:    shortfall = shortfallContractAddress_;
119:    emit ShortfallContractUpdated(oldShortfallContractAddress, shortfallContractAddress_);

129:    pancakeSwapRouter = pancakeSwapRouter_;
130:    emit PancakeSwapRouterUpdated(oldPancakeSwapRouter, pancakeSwapRouter_);

141:    minAmountToConvert = minAmountToConvert_;
142:    emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
```

#### [G-06] Remove unnecessary variables

Removing unnecessary variables that do not aid readability can make the code simpler and save gas.

1. [Comptroller.sol#L163](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L163)

   ```diff
           for (uint256 i; i < len; ++i) {
   -            VToken vToken = VToken(vTokens[i]);
   -
   -            _addToMarket(vToken, msg.sender);
   +            _addToMarket(VToken(vTokens[i]), msg.sender);
               results[i] = NO_ERROR;
           }
   ```

1. [Comptroller.sol#L940](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L940)

   ```diff
   -        uint256 rewardsDistributorsLen = rewardsDistributors.length;
   -        _ensureMaxLoops(rewardsDistributorsLen + 1);
   +        _ensureMaxLoops(rewardsDistributors.length + 1);
   ```

1. [Comptroller.sol#L1123](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1123)

   ```diff
   -            address rewardToken = address(rewardsDistributors[i].rewardToken());
                rewardSpeeds[i] = RewardSpeeds({
   -                rewardToken: rewardToken,
   +                rewardToken: address(rewardsDistributors[i].rewardToken()),
                    supplySpeed: rewardsDistributors[i].rewardTokenSupplySpeeds(vToken),
                    borrowSpeed: rewardsDistributors[i].rewardTokenBorrowSpeeds(vToken)
                });
   ```

1. [Comptroller.sol#L1308](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1308)

   ```diff
   -            VToken asset = assets[i];
   -
                // Read the balances and exchange rate from the vToken
                (uint256 vTokenBalance, uint256 borrowBalance, uint256 exchangeRateMantissa) = _safeGetAccountSnapshot(
   -                asset,
   +                assets[i],
                    account
                );

   ```
