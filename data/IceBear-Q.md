# Venus Protocol
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [N-1](#N-1) | Use of block.timestamp | 2 |
| [N-2](#N-2) | Critical Changes Should Use Two-step Procedure | 17 |
| [N-3](#N-3) | Add a timelock to critical functions | 17 |
| [N-4](#N-4) | Lack of NatSpec | 17 |
| [N-5](#N-5) | Initial value check is missing in Set Functions | 31 |
| [N-6](#N-6) | Large multiples of ten should use scientific notation rather than decimal literals, for readability | 6 |
| [N-7](#N-7) | Event is missing `indexed` fields  | 59 |
| [N-8](#N-8) | Unused imports | 5 |
| [N-9](#N-9) | Use require instead of assert | 1 |
| [N-10](#N-10) | Functions not used internally could be marked external  | 5 |
| [N-11](#N-11) | Constant values such as a call to keccak256(), should used to immutable rather than constant | 2 |


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Misplacement of code comments  | 1 |
| [L-2](#L-2) | Do not use deprecated library functions  | 4 |
| [L-3](#L-3) | Initializers could be front-run  | 24 |
| [L-4](#L-4) | Loss of precision | 24 |
| [L-5](#L-5) | Upgradeable contract not initialized | 2 |
| [L-6](#L-6) | Contracts are not using their OZ upgradeable counterparts | 5 |

### [N-1] Use of block.timestamp
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
#### Recommended Mitigation Steps
Block timestamps should not be used for entropy or generating random numbers — i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

Instances where block.timestamp is used:

*Find (2) instance(s) in contracts*:
```solidity
File: Pool/PoolRegistry.sol

401:         VenusPool memory pool = VenusPool(name, msg.sender, comptroller, block.number, block.timestamp);

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: RiskFund/RiskFund.sol

265:                         block.timestamp

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

### [N-2] Critical Changes Should Use Two-step Procedure

  The critical procedures should be two step process.

*Find (17) instance(s) in contracts*:
```solidity
File: Comptroller.sol

961:     function setPriceOracle(PriceOracle newOracle) external onlyOwner {

973:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

198:     function setShortfallContract(Shortfall shortfall_) external onlyOwner {

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

171:     function updateRewardTokenSupplyIndex(address vToken) external onlyComptroller {

187:     function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {

249:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/RiskFund.sol

99:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

110:     function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {

126:     function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {

205:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

348:     function updatePoolRegistry(address _poolRegistry) external onlyOwner {

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

330:     function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

515:     function setShortfallContract(address shortfall_) external onlyOwner {

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [N-3] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

*Find (17) instance(s) in contracts*:
```solidity
File: Comptroller.sol

961:     function setPriceOracle(PriceOracle newOracle) external onlyOwner {

973:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

198:     function setShortfallContract(Shortfall shortfall_) external onlyOwner {

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

171:     function updateRewardTokenSupplyIndex(address vToken) external onlyComptroller {

187:     function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {

249:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/RiskFund.sol

99:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

110:     function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {

126:     function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {

205:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

348:     function updatePoolRegistry(address _poolRegistry) external onlyOwner {

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

330:     function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

515:     function setShortfallContract(address shortfall_) external onlyOwner {

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [N-4] Lack of NatSpec
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### Recommendation
For Solidity you may choose /// for single or multi-line comments, or /** and ending with */.

Recommendation Code Style: (from Uniswap3)

```solidity
/// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
/// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
/// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
/// on tickLower, tickUpper, the amount of liquidity, and the current price.
/// @param recipient The address for which the liquidity will be created
/// @param tickLower The lower tick of the position in which to add liquidity
/// @param tickUpper The upper tick of the position in which to add liquidity
/// @param amount The amount of liquidity to mint
/// @param data Any data that should be passed through to the callback
/// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
/// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
function mint(
    address recipient,
    int24 tickLower,
    int24 tickUpper,
    uint128 amount,
    bytes calldata data
) external returns (uint256 amount0, uint256 amount1);

```

*Find (17) instance(s) in contracts*:
```solidity
File: ComptrollerInterface.sol

The issus is throughout the contract.

```
[ComptrollerInterface.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerInterface.sol)

### [N-5] Initial value check is missing in Set Functions
A check regarding whether the current value and the new value are the same should be added.

*Find (31) instance(s) in contracts*:
```solidity
File: BaseJumpRateModelV2.sol

88:     function updateJumpRateModel(
            uint256 baseRatePerYear,
            uint256 multiplierPerYear,
            uint256 jumpMultiplierPerYear,
            uint256 kink_
        ) external virtual {
            string memory signature = "updateJumpRateModel(uint256,uint256,uint256,uint256)";
            bool isAllowedToCall = accessControlManager.isAllowedToCall(msg.sender, signature);
    
            if (!isAllowedToCall) {
                revert Unauthorized(msg.sender, address(this), signature);
            }
    
            _updateJumpRateModel(baseRatePerYear, multiplierPerYear, jumpMultiplierPerYear, kink_);
        }

```
[BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol)

```solidity
File: Comptroller.sol

702:     function setCloseFactor(uint256 newCloseFactorMantissa) external {
             _checkAccessAllowed("setCloseFactor(uint256)");
             require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");
             require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");
     
             uint256 oldCloseFactorMantissa = closeFactorMantissa;
             closeFactorMantissa = newCloseFactorMantissa;
             emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
         }

779:     function setLiquidationIncentive(uint256 newLiquidationIncentiveMantissa) external {
             require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");
     
             _checkAccessAllowed("setLiquidationIncentive(uint256)");
     
             // Save current value for use in log
             uint256 oldLiquidationIncentiveMantissa = liquidationIncentiveMantissa;
     
             // Set liquidation incentive to new incentive
             liquidationIncentiveMantissa = newLiquidationIncentiveMantissa;
     
             // Emit event with old incentive, new incentive
             emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);
         }

836:     function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {
             _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");
     
             uint256 numMarkets = vTokens.length;
             uint256 numBorrowCaps = newBorrowCaps.length;
     
             require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");
     
             _ensureMaxLoops(numMarkets);
     
             for (uint256 i; i < numMarkets; ++i) {
                 borrowCaps[address(vTokens[i])] = newBorrowCaps[i];
                 emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);
             }
         }

885:     function setActionsPaused(
             VToken[] calldata marketsList,
             Action[] calldata actionsList,
             bool paused
         ) external {
             _checkAccessAllowed("setActionsPaused(address[],uint256[],bool)");
     
             uint256 marketsCount = marketsList.length;
             uint256 actionsCount = actionsList.length;
     
             _ensureMaxLoops(marketsCount);
     
             for (uint256 marketIdx; marketIdx < marketsCount; ++marketIdx) {
                 for (uint256 actionIdx; actionIdx < actionsCount; ++actionIdx) {
                     _setActionPaused(address(marketsList[marketIdx]), actionsList[actionIdx], paused);
                 }
             }
         }

912:     function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {
             _checkAccessAllowed("setMinLiquidatableCollateral(uint256)");
     
             uint256 oldMinLiquidatableCollateral = minLiquidatableCollateral;
             minLiquidatableCollateral = newMinLiquidatableCollateral;
             emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);
         }

973:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {
             _setMaxLoopsLimit(limit);
         }

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Lens/PoolLens.sol

453:     function updateMarketBorrowIndex(
             address vToken,
             RewardsDistributor rewardsDistributor,
             RewardTokenState memory borrowState,
             Exp memory marketBorrowIndex
         ) internal view {
             uint256 borrowSpeed = rewardsDistributor.rewardTokenBorrowSpeeds(vToken);
             uint256 blockNumber = block.number;
             uint256 deltaBlocks = sub_(blockNumber, uint256(borrowState.block));
             if (deltaBlocks > 0 && borrowSpeed > 0) {
                 // Remove the total earned interest rate since the opening of the market from total borrows
                 uint256 borrowAmount = div_(VToken(vToken).totalBorrows(), marketBorrowIndex);
                 uint256 tokensAccrued = mul_(deltaBlocks, borrowSpeed);
                 Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });
                 Double memory index = add_(Double({ mantissa: borrowState.index }), ratio);
                 borrowState.index = safe224(index.mantissa, "new index overflows");
                 borrowState.block = safe32(blockNumber, "block number overflows");
             } else if (deltaBlocks > 0) {
                 borrowState.block = safe32(blockNumber, "block number overflows");
             }
         }

475:     function updateMarketSupplyIndex(
             address vToken,
             RewardsDistributor rewardsDistributor,
             RewardTokenState memory supplyState
         ) internal view {
             uint256 supplySpeed = rewardsDistributor.rewardTokenSupplySpeeds(vToken);
             uint256 blockNumber = block.number;
             uint256 deltaBlocks = sub_(blockNumber, uint256(supplyState.block));
             if (deltaBlocks > 0 && supplySpeed > 0) {
                 uint256 supplyTokens = VToken(vToken).totalSupply();
                 uint256 tokensAccrued = mul_(deltaBlocks, supplySpeed);
                 Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });
                 Double memory index = add_(Double({ mantissa: supplyState.index }), ratio);
                 supplyState.index = safe224(index.mantissa, "new index overflows");
                 supplyState.block = safe32(blockNumber, "block number overflows");
             } else if (deltaBlocks > 0) {
                 supplyState.block = safe32(blockNumber, "block number overflows");
             }
         }

```
[Lens/PoolLens.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol)

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {
             _setProtocolShareReserve(protocolShareReserve_);
         }

198:     function setShortfallContract(Shortfall shortfall_) external onlyOwner {
             _setShortfallContract(shortfall_);
         }

332:     function setPoolName(address comptroller, string calldata name) external {
             _checkAccessAllowed("setPoolName(address,string)");
             _ensureValidName(name);
             string memory oldName = _poolByComptroller[comptroller].name;
             _poolByComptroller[comptroller].name = name;
             emit PoolNameSet(comptroller, oldName, name);
         }

343:     function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {
             _checkAccessAllowed("updatePoolMetadata(address,VenusPoolMetaData)");
             VenusPoolMetaData memory oldMetadata = metadata[comptroller];
             metadata[comptroller] = _metadata;
             emit PoolMetadataUpdated(comptroller, oldMetadata, _metadata);
         }

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

171:     function updateRewardTokenSupplyIndex(address vToken) external onlyComptroller {
             _updateRewardTokenSupplyIndex(vToken);
         }

187:     function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {
             _updateRewardTokenBorrowIndex(vToken, marketBorrowIndex);
         }

197:     function setRewardTokenSpeeds(
             VToken[] memory vTokens,
             uint256[] memory supplySpeeds,
             uint256[] memory borrowSpeeds
         ) external {
             _checkAccessAllowed("setRewardTokenSpeeds(address[],uint256[],uint256[])");
             uint256 numTokens = vTokens.length;
             require(
                 numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,
                 "RewardsDistributor::setRewardTokenSpeeds invalid input"
             );
     
             for (uint256 i; i < numTokens; ++i) {
                 _setRewardTokenSpeed(vTokens[i], supplySpeeds[i], borrowSpeeds[i]);
             }
         }

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {
             // note that REWARD TOKEN speed could be set to 0 to halt liquidity rewards for a contributor
             updateContributorRewards(contributor);
             if (rewardTokenSpeed == 0) {
                 // release storage
                 delete lastContributorBlock[contributor];
             } else {
                 lastContributorBlock[contributor] = getBlockNumber();
             }
             rewardTokenContributorSpeeds[contributor] = rewardTokenSpeed;
     
             emit ContributorRewardTokenSpeedUpdated(contributor, rewardTokenSpeed);
         }

249:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {
             _setMaxLoopsLimit(limit);
         }

257:     function updateContributorRewards(address contributor) public {
             uint256 rewardTokenSpeed = rewardTokenContributorSpeeds[contributor];
             uint256 blockNumber = getBlockNumber();
             uint256 deltaBlocks = sub_(blockNumber, lastContributorBlock[contributor]);
             if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
                 uint256 newAccrued = mul_(deltaBlocks, rewardTokenSpeed);
                 uint256 contributorAccrued = add_(rewardTokenAccrued[contributor], newAccrued);
     
                 rewardTokenAccrued[contributor] = contributorAccrued;
                 lastContributorBlock[contributor] = blockNumber;
     
                 emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
             }
         }

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

97:     function updateAssetsState(address comptroller, address asset)
            public
            override(IProtocolShareReserve, ReserveHelpers)
        {
            super.updateAssetsState(comptroller, asset);
        }

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/ReserveHelpers.sol

50:     function updateAssetsState(address comptroller, address asset) public virtual {
            require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
            require(asset != address(0), "ReserveHelpers: Asset address invalid");
            require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
            require(
                PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
                "ReserveHelpers: The pool doesn't support the asset"
            );
    
            uint256 currentBalance = IERC20Upgradeable(asset).balanceOf(address(this));
            uint256 assetReserve = assetsReserves[asset];
            if (currentBalance > assetReserve) {
                uint256 balanceDifference;
                unchecked {
                    balanceDifference = currentBalance - assetReserve;
                }
                assetsReserves[asset] += balanceDifference;
                poolsAssetsReserves[comptroller][asset] += balanceDifference;
                emit AssetsReservesUpdated(comptroller, asset, balanceDifference);
            }
        }

```
[RiskFund/ReserveHelpers.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol)

```solidity
File: RiskFund/RiskFund.sol

137:     function setMinAmountToConvert(uint256 minAmountToConvert_) external {
             _checkAccessAllowed("setMinAmountToConvert(uint256)");
             require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
             uint256 oldMinAmountToConvert = minAmountToConvert;
             minAmountToConvert = minAmountToConvert_;
             emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
         }

205:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {
             _setMaxLoopsLimit(limit);
         }

214:     function updateAssetsState(address comptroller, address asset) public override(IRiskFund, ReserveHelpers) {
             super.updateAssetsState(comptroller, asset);
         }

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

321:     function updateMinimumPoolBadDebt(uint256 _minimumPoolBadDebt) external {
             _checkAccessAllowed("updateMinimumPoolBadDebt(uint256)");
             uint256 oldMinimumPoolBadDebt = minimumPoolBadDebt;
             minimumPoolBadDebt = _minimumPoolBadDebt;
             emit MinimumPoolBadDebtUpdated(oldMinimumPoolBadDebt, _minimumPoolBadDebt);
         }

334:     function updateWaitForFirstBidder(uint256 _waitForFirstBidder) external {
             _checkAccessAllowed("updateWaitForFirstBidder(uint256)");
             uint256 oldWaitForFirstBidder = waitForFirstBidder;
             waitForFirstBidder = _waitForFirstBidder;
             emit WaitForFirstBidderUpdated(oldWaitForFirstBidder, _waitForFirstBidder);
         }

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

310:     function setProtocolSeizeShare(uint256 newProtocolSeizeShareMantissa_) external {
             _checkAccessAllowed("setProtocolSeizeShare(uint256)");
             uint256 liquidationIncentive = ComptrollerViewInterface(address(comptroller)).liquidationIncentiveMantissa();
             if (newProtocolSeizeShareMantissa_ + 1e18 > liquidationIncentive) {
                 revert ProtocolSeizeShareTooBig();
             }
     
             uint256 oldProtocolSeizeShareMantissa = protocolSeizeShareMantissa;
             protocolSeizeShareMantissa = newProtocolSeizeShareMantissa_;
             emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);
         }

330:     function setReserveFactor(uint256 newReserveFactorMantissa) external override nonReentrant {
             _checkAccessAllowed("setReserveFactor(uint256)");
     
             accrueInterest();
             _setReserveFactorFresh(newReserveFactorMantissa);
         }

369:     function setInterestRateModel(InterestRateModel newInterestRateModel) external override {
             _checkAccessAllowed("setInterestRateModel(address)");
     
             accrueInterest();
             _setInterestRateModelFresh(newInterestRateModel);
         }

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {
             _setProtocolShareReserve(protocolShareReserve_);
         }

515:     function setShortfallContract(address shortfall_) external onlyOwner {
             _setShortfallContract(shortfall_);
         }

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [N-6] Large multiples of ten should use scientific notation rather than decimal literals, for readability
#### Recommendation
Scientific notation should be used for better code readability.
Use scientific notation (e.g. 1e18, 1e6) rather than exponentiation (e.g. 10**18, 100000).

*Find (6) instance(s) in contracts*:
```solidity
File: BaseJumpRateModelV2.sol

23:     uint256 public constant blocksPerYear = 10512000;

```
[BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol)

```solidity
File: ExponentialNoError.sol

64:         require(n < 2**224, errorMessage);

69:         require(n < 2**32, errorMessage);

```
[ExponentialNoError.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol)

```solidity
File: Shortfall/Shortfall.sol

60:     uint256 private constant MAX_BPS = 10000;

149:         incentiveBps = 1000;

163:         require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

### [N-7] Event is missing `indexed` fields 
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Find (59) instance(s) in contracts*:
```solidity
File: BaseJumpRateModelV2.sol

45:     event NewInterestParams(

```
[BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol)

```solidity
File: Comptroller.sol

30:     event MarketEntered(VToken vToken, address account);

33:     event MarketExited(VToken vToken, address account);

36:     event NewCloseFactor(uint256 oldCloseFactorMantissa, uint256 newCloseFactorMantissa);

39:     event NewCollateralFactor(VToken vToken, uint256 oldCollateralFactorMantissa, uint256 newCollateralFactorMantissa);

42:     event NewLiquidationThreshold(

49:     event NewLiquidationIncentive(uint256 oldLiquidationIncentiveMantissa, uint256 newLiquidationIncentiveMantissa);

52:     event NewPriceOracle(PriceOracle oldPriceOracle, PriceOracle newPriceOracle);

55:     event ActionPausedMarket(VToken vToken, Action action, bool pauseState);

58:     event NewBorrowCap(VToken indexed vToken, uint256 newBorrowCap);

61:     event NewMinLiquidatableCollateral(uint256 oldMinLiquidatableCollateral, uint256 newMinLiquidatableCollateral);

64:     event NewSupplyCap(VToken indexed vToken, uint256 newSupplyCap);

70:     event MarketSupported(VToken vToken);

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Factories/VTokenProxyFactory.sol

26:     event VTokenProxyDeployed(VTokenArgs args);

```
[Factories/VTokenProxyFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol)

```solidity
File: MaxLoopsLimitHelper.sol

16:     event MaxLoopsLimitUpdated(uint256 oldMaxLoopsLimit, uint256 newmaxLoopsLimit);

```
[MaxLoopsLimitHelper.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol)

```solidity
File: Pool/PoolRegistry.sol

113:     event PoolRegistered(address indexed comptroller, VenusPool pool);

118:     event PoolNameSet(address indexed comptroller, string oldName, string newName);

123:     event PoolMetadataUpdated(

132:     event MarketAdded(address indexed comptroller, address vTokenAddress);

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

57:     event DistributedSupplierRewardToken(

66:     event DistributedBorrowerRewardToken(

75:     event RewardTokenSupplySpeedUpdated(VToken indexed vToken, uint256 newSpeed);

78:     event RewardTokenBorrowSpeedUpdated(VToken indexed vToken, uint256 newSpeed);

81:     event RewardTokenGranted(address recipient, uint256 amount);

84:     event ContributorRewardTokenSpeedUpdated(address indexed contributor, uint256 newSpeed);

87:     event MarketInitialized(address vToken);

90:     event RewardTokenSupplyIndexUpdated(address vToken);

93:     event RewardTokenBorrowIndexUpdated(address vToken, Exp marketBorrowIndex);

96:     event ContributorRewardsUpdated(address contributor, uint256 rewardAccrued);

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

22:     event FundsReleased(address comptroller, address asset, uint256 amount);

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/ReserveHelpers.sol

30:     event AssetsReservesUpdated(address indexed comptroller, address indexed asset, uint256 amount);

```
[RiskFund/ReserveHelpers.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol)

```solidity
File: RiskFund/RiskFund.sol

47:     event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);

50:     event MinAmountToConvertUpdated(uint256 oldMinAmountToConvert, uint256 newMinAmountToConvert);

53:     event SwappedPoolsAssets(address[] markets, uint256[] amountsOutMin, uint256 totalAmount);

56:     event TransferredReserveForAuction(address comptroller, uint256 amount);

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

75:     event AuctionStarted(

86:     event BidPlaced(address indexed comptroller, uint256 auctionStartBlock, uint256 bidBps, address indexed bidder);

89:     event AuctionClosed(

100:     event AuctionRestarted(address indexed comptroller, uint256 auctionStartBlock);

106:     event MinimumPoolBadDebtUpdated(uint256 oldMinimumPoolBadDebt, uint256 newMinimumPoolBadDebt);

109:     event WaitForFirstBidderUpdated(uint256 oldWaitForFirstBidder, uint256 newWaitForFirstBidder);

112:     event NextBidderBlockLimitUpdated(uint256 oldNextBidderBlockLimit, uint256 newNextBidderBlockLimit);

115:     event IncentiveBpsUpdated(uint256 oldIncentiveBps, uint256 newIncentiveBps);

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VTokenInterfaces.sol

149:     event AccrueInterest(uint256 cashPrior, uint256 interestAccumulated, uint256 borrowIndex, uint256 totalBorrows);

154:     event Mint(address indexed minter, uint256 mintAmount, uint256 mintTokens, uint256 accountBalance);

159:     event Redeem(address indexed redeemer, uint256 redeemAmount, uint256 redeemTokens, uint256 accountBalance);

164:     event Borrow(address indexed borrower, uint256 borrowAmount, uint256 accountBorrows, uint256 totalBorrows);

169:     event RepayBorrow(

184:     event BadDebtIncreased(address indexed borrower, uint256 badDebtDelta, uint256 badDebtOld, uint256 badDebtNew);

191:     event BadDebtRecovered(uint256 badDebtOld, uint256 badDebtNew);

232:     event NewProtocolSeizeShare(uint256 oldProtocolSeizeShareMantissa, uint256 newProtocolSeizeShareMantissa);

237:     event NewReserveFactor(uint256 oldReserveFactorMantissa, uint256 newReserveFactorMantissa);

242:     event ReservesAdded(address indexed benefactor, uint256 addAmount, uint256 newTotalReserves);

247:     event ReservesReduced(address indexed admin, uint256 reduceAmount, uint256 newTotalReserves);

252:     event Transfer(address indexed from, address indexed to, uint256 amount);

257:     event Approval(address indexed owner, address indexed spender, uint256 amount);

262:     event HealBorrow(address payer, address borrower, uint256 repayAmount);

267:     event SweepToken(address token);

```
[VTokenInterfaces.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol)

```solidity
File: WhitePaperInterestRateModel.sol

29:     event NewInterestParams(uint256 baseRatePerBlock, uint256 multiplierPerBlock);

```
[WhitePaperInterestRateModel.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol)

### [N-8] Unused imports

*Find (5) instance(s) in contracts*:
```solidity
File: Factories/VTokenProxyFactory.sol

8: import "../VTokenInterfaces.sol";

```
[Factories/VTokenProxyFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol)

```solidity
File: Pool/PoolRegistry.sol

18: import "../VTokenInterfaces.sol";

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: VToken.sol

8: import "./VTokenInterfaces.sol";

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

```solidity
File: VTokenInterfaces.sol

5: import "@venusprotocol/oracle/contracts/PriceOracle.sol";

8: import "./ErrorReporter.sol";

```
[VTokenInterfaces.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol)

### [N-9] Use require instead of assert
Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

`assert() and reqire();`

The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay.This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.

*Find (1) instance(s) in contracts*:
```solidity
File: Comptroller.sol

225:         assert(assetIndex < len);

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

### [N-10] Functions not used internally could be marked external 

*Find (5) instance(s) in contracts*:
```solidity
File: Comptroller.sol

1144:     function getRewardDistributors() public view returns (RewardsDistributor[] memory) {

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

97:     function updateAssetsState(address comptroller, address asset)

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/RiskFund.sol

214:     function updateAssetsState(address comptroller, address asset) public override(IRiskFund, ReserveHelpers) {

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: VToken.sol

625:     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

```solidity
File: WhitePaperInterestRateModel.sol

67:     function getSupplyRate(

```
[WhitePaperInterestRateModel.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol)

### [N-11] Constant values such as a call to keccak256(), should used to immutable rather than constant
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
  
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Find (2) instance(s) in contracts*:
```solidity
File: ExponentialNoError.sol

22:     uint256 internal constant halfExpScale = expScale / 2;

23:     uint256 internal constant mantissaOne = expScale;

```
[ExponentialNoError.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol)


### [L-1] Misplacement of code comments
The comment on L836 should be placed before L805.

*Find (1) instance(s) in contracts*:
```solidity
File: VToken.sol

836:     // Require tokens is zero or amount is also zero

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [L-2] Do not use deprecated library functions 

*Find (4) instance(s) in contracts*:
```solidity
File: Pool/PoolRegistry.sol

322:         token.safeApprove(address(vToken), 0);

323:         token.safeApprove(address(vToken), amountToSupply);

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: RiskFund/RiskFund.sol

258:                     IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);

259:                     IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

### [L-3] Initializers could be front-run 
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Find (24) instance(s) in contracts*:
```solidity
File: Comptroller.sol

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {

139:         __Ownable2Step_init();

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Pool/PoolRegistry.sol

164:     function initialize(

171:     ) external initializer {

172:         __Ownable2Step_init();

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

111:     function initialize(

116:     ) external initializer {

119:         __Ownable2Step_init();

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

39:     function initialize(address _protocolIncome, address _riskFund) external initializer {

43:         __Ownable2Step_init();

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/RiskFund.sol

73:     function initialize(

79:     ) external initializer {

85:         __Ownable2Step_init();

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

131:     function initialize(

136:     ) external initializer {

141:         __Ownable2Step_init();

143:         __ReentrancyGuard_init();

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

59:     function initialize(

71:     ) external initializer {

75:         _initialize(

1350:     function _initialize(

1363:         __Ownable2Step_init();

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [L-4] Loss of precision

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator.

*Find (24) instance(s) in contracts*:
```solidity
File: BaseJumpRateModelV2.sol

120:         uint256 rateToPool = (borrowRate * oneMinusReserveFactor) / BASE;

121:         return (utilizationRate(cash, borrows, reserves) * rateToPool) / BASE;

157:         baseRatePerBlock = baseRatePerYear / blocksPerYear;

159:         jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;

180:             return ((util * multiplierPerBlock) / BASE) + baseRatePerBlock;

182:             uint256 normalRate = ((kink * multiplierPerBlock) / BASE) + baseRatePerBlock;

187:             return ((excessUtil * jumpMultiplierPerBlock) / BASE) + normalRate;

```
[BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol)

```solidity
File: ExponentialNoError.sol

31:         return exp.mantissa / expScale;

98:         return Exp({ mantissa: mul_(a.mantissa, b.mantissa) / expScale });

106:         return mul_(a, b.mantissa) / expScale;

110:         return Double({ mantissa: mul_(a.mantissa, b.mantissa) / doubleScale });

118:         return mul_(a, b.mantissa) / doubleScale;

150:         return a / b;

```
[ExponentialNoError.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol)

```solidity
File: Shortfall/Shortfall.sol

181:                     uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /

186:                 uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);

228:                 uint256 bidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) / MAX_BPS);

244:             riskFundBidAmount = (auction.seizedRiskFund * auction.highestBidBps) / MAX_BPS;

405:         uint256 incentivizedRiskFundBalance = poolBadDebt + ((poolBadDebt * incentiveBps) / MAX_BPS);

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

1478:             uint256 exchangeRate = (cashPlusBorrowsMinusReserves * expScale) / _totalSupply;

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

```solidity
File: WhitePaperInterestRateModel.sol

37:         baseRatePerBlock = baseRatePerYear / blocksPerYear;

38:         multiplierPerBlock = multiplierPerYear / blocksPerYear;

56:         return ((ur * multiplierPerBlock) / BASE) + baseRatePerBlock;

75:         uint256 rateToPool = (borrowRate * oneMinusReserveFactor) / BASE;

76:         return (utilizationRate(cash, borrows, reserves) * rateToPool) / BASE;

```
[WhitePaperInterestRateModel.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol)

### [L-5] Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user.

*Find (2) instance(s) in contracts*:
```solidity
File: VToken.sol

//@audit missing __Ownable2Step_init()
19: contract VToken is

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)

### [L-6] Contracts are not using their OZ upgradeable counterparts
The non-upgradeable standard version of OpenZeppelin’s library, such as Ownable, Pausable, Address, Context, SafeERC20, ERC1967Upgrade etc, are inherited / used by both the proxy and the implementation contracts.

#### Recommendation
Where applicable, use the contracts from @openzeppelin/contracts-upgradeable instead of @openzeppelin/contracts.

*Find (5) instance(s) in contracts*:
```solidity
File: Factories/VTokenProxyFactory.sol

4: import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";

```
[Factories/VTokenProxyFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol)

```solidity
File: Lens/PoolLens.sol

4: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

```
[Lens/PoolLens.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol)

```solidity
File: Pool/PoolRegistry.sol

6: import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";

7: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Proxy/UpgradeableBeacon.sol

4: import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";

```
[Proxy/UpgradeableBeacon.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners  | 16 |
### [M-1] Centralization Risk for trusted owners 

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Find (16) instance(s) in contracts*:
```solidity
File: Comptroller.sol

927:     function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner {

961:     function setPriceOracle(PriceOracle newOracle) external onlyOwner {

973:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)

```solidity
File: Pool/PoolRegistry.sol

188:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

198:     function setShortfallContract(Shortfall shortfall_) external onlyOwner {

```
[Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)

```solidity
File: Rewards/RewardsDistributor.sol

181:     function grantRewardToken(address recipient, uint256 amount) external onlyOwner {

219:     function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external onlyOwner {

249:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)

```solidity
File: RiskFund/ProtocolShareReserve.sol

53:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

```
[RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)

```solidity
File: RiskFund/RiskFund.sol

99:     function setPoolRegistry(address _poolRegistry) external onlyOwner {

110:     function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {

126:     function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {

205:     function setMaxLoopsLimit(uint256 limit) external onlyOwner {

```
[RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)

```solidity
File: Shortfall/Shortfall.sol

348:     function updatePoolRegistry(address _poolRegistry) external onlyOwner {

```
[Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)

```solidity
File: VToken.sol

505:     function setProtocolShareReserve(address payable protocolShareReserve_) external onlyOwner {

515:     function setShortfallContract(address shortfall_) external onlyOwner {

```
[VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)
