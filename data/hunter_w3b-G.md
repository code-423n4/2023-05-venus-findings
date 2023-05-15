## Summary

### Gas Optimizations

|               | Issue                                                                                       | Instances | Total Gas Saved |
| ------------- | :------------------------------------------------------------------------------------------ | :-------: | :-------------: |
| [G&#x2011;01] | Avoid `contract existence` checks by using solidity version 0.8.10 or later                 |    32     |      3200       |
| [G&#x2011;02] | Use solidity version `0.8.19` to gain some gas boost                                        |    28     |      2464       |
| [G&#x2011;03] |  Massive 15k per tx gas savings - use 1 and 2 for `Reentrancy guard`                        |     1     |      15000      |
| [G&#x2011;04] |  `<X> += <Y>` costes more gas than`<X> = <X> + <Y>` for state variables                     |     2     |       226       |
| [G&#x2011;05] | Not using the named return variables when a function `returns` , wastes deployment gas      |    10     |                 |
| [G&#x2011;06] | Can make the `variable` outside the loop to save gas                                        |    18     |        -        |
| [G&#x2011;07] | Using `calldata` instead of `memory` for read-only arguments in external functions save gas |     6     |       720       |
| [G&#x2011;08] | Use `assembly` to check for `address(0)`                                                    |    45     |        -        |
| [G&#x2011;09] | State variables only set in the constructor should be declared `immutable`                  |     1     |      2000       |
| [G&#x2011;10] | Use in constants instead of `type(uintx).max`                                               |     5     |        -        |
| [G&#x2011;11] | Duplicated `if()` checks should be refactored to a modifier or function                     |    11     |        -        |
| [G&#x2011;12] | Use `nested if` and, avoid multiple check `combinations` &&                                 |    12     |       36        |
| [G&#x2011;13] | Use  `assembly` to write address storage values                                             |     7     |        -        |
| [G&#x2011;14] | Do not calculate `constants`                                                                |     1     |        -        |
| [G&#x2011;15] | Use hardcode address instead `address(this)`                                                |    30     |        -        |
| [G&#x2011;16] | Use `!= 0` instead of `> 0` for unsigned integer comparison                                 |     6     |        -        |
| [G&#x2011;17] | Use `uint256(1)/uint256(2)` instead for `true` and `false` boolean states                   |     3     |      15000      |
| [G&#x2011;18] | Shorten the `array` rather than copying to a `new` one                                      |     3     |        -        |
| [G&#x2011;19] | Usage of `uints/ints` smaller than `32 BYTES` 256bits incurs overhead                       |     7     |        -        |

Total: 228 instances over 19 issues gas save ~38646

## Gas Optimizations

## [G-01]  Avoid `contract existence` checks by using solidity version 0.8.10 or later

### Summary

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

### Details

There are **32** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

/// @audit totalSupply()
263     VToken(vToken).totalSupply();

/// @audit exchangeRateStored()
264     Exp({ mantissa: VToken(vToken).exchangeRateStored() });

/// @audit totalBorrows()
352            uint256 totalBorrows = VToken(vToken).totalBorrows();

/// @audit  borrowIndex()
371     Exp({ mantissa: VToken(vToken).borrowIndex() });

/// @audit  borrowIndex()
403     Exp({ mantissa: VToken(vToken).borrowIndex() });

/// @audit  borrowBalanceStored()
446        uint256 borrowBalance = VToken(vTokenBorrowed).borrowBalanceStored(borrower);

/// @audit   comptroller()
504            if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {


/// @audit   comptroller()
513            if (VToken(vTokenCollateral).comptroller() != VToken(seizerContract).comptroller()) {

/// @audit   exchangeRateStored()
1099        uint256 exchangeRateMantissa = VToken(vTokenCollateral).exchangeRateStored();
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

/// @audit  updateAssetsState()
1233        IProtocolShareReserve(protocolShareReserve).updateAssetsState(address(comptroller), underlying);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1233

```solidity
File: /contracts/Lens/PoolLens.sol

/// @audit  getAllMarkets()
225        VToken[] memory markets = ComptrollerInterface(comptrollerAddress).getAllMarkets();

/// @audit  badDebt()
267                VToken(address(markets[i])).badDebt() *

/// @audit  decimals()
361        underlyingDecimals = IERC20Metadata(vToken.underlying()).decimals();

/// @audit  totalBorrows()
464            uint256 borrowAmount = div_(VToken(vToken).totalBorrows(), marketBorrowIndex);

/// @audit  totalSupply()
484            uint256 supplyTokens = VToken(vToken).totalSupply();

/// @audit  borrowBalanceStored()
511        uint256 borrowerAmount = div_(VToken(vToken).borrowBalanceStored(borrower), marketBorrowIndex);

/// @audit  balanceOf()
531        uint256 supplierTokens = VToken(vToken).balanceOf(supplier);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

```solidity
File: /contracts/Shortfall/Shortfall.sol

/// @audit   getPoolByComptroller()
360        PoolRegistryInterface.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);

/// @audit  getAllMarkets()
451        return ComptrollerInterface(comptroller).getAllMarkets();
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

/// @audit   balanceOf()
358        uint256 supplierTokens = VToken(vToken).balanceOf(supplier);

/// @audit   borrowBalanceStored()
396        uint256 borrowerAmount = div_(VToken(vToken).borrowBalanceStored(borrower), marketBorrowIndex);

/// @audit   totalSupply()
436            uint256 supplyTokens = VToken(vToken).totalSupply();

/// @audit   totalBorrows()
464            uint256 borrowAmount = div_(VToken(vToken).totalBorrows(), marketBorrowIndex);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

```solidity
File: /contracts/Pool/PoolRegistry.sol

/// @audit    deploy()
270     InterestRateModel(
                   jumpRateFactory.deploy(

/// @audit      decimals()
284          uint256 underlyingDecimals = IERC20Metadata(input.asset).decimals();
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

```solidity
File: /contracts/RiskFund/RiskFund.sol

/// @audit    convertibleBaseAsset()
113            IShortfall(shortfallContractAddress_).convertibleBaseAsset() == convertibleBaseAsset,

/// @audit    getPoolByComptroller()
170            PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);

/// @audit     swapExactTokensForTokens()
260                    uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

```solidity
File: /contracts/RiskFund/ProtocolShareReserve.sol

/// @audit   updateAssetsState()
85        IRiskFund(riskFund).updateAssetsState(comptroller, asset);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L85

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

/// @audit     isComptroller()
39        require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");

/// @audit      isComptroller()
51        require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");

/// @audit      getVTokenForAsset()
55            PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol

## [G-02]  Use solidity version `0.8.19` to gain some gas boost

### Summary

Upgrade to the latest solidity version 0.8.19 to get additional gas savings. See latest release for

reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

### Details

There are **28** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

2     pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L2

```solidity
File: /contracts/VToken.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L2

```solidity
File: /contracts/Lens/PoolLens.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L2

```solidity
File: /contracts/Shortfall/Shortfall.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L2

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L2

```solidity
File: /contracts/Pool/PoolRegistry.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L2

```solidity
File: /contracts/RiskFund/RiskFund.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L2

```solidity
File: /contracts/VTokenInterfaces.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main #L2

```solidity
File: /contracts/ExponentialNoError.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L2

```solidity
File: /contracts/BaseJumpRateModelV2.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L2

```solidity
File: /contracts/ComptrollerInterface.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerInterface.sol#L2

```solidity
File: /contracts/ComptrollerStorage.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L2

```solidity
File: /contracts/RiskFund/ProtocolShareReserve.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L2

```solidity
File: /contracts/Factories/VTokenProxyFactory.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L2

```solidity
File: /contracts/WhitePaperInterestRateModel.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L2

```solidity
File: /contracts/ErrorReporter.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L2

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L2

```solidity
File: /contracts/JumpRateModelV2.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L2

```solidity
File: /contracts/Factories/JumpRateModelFactory.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/JumpRateModelFactory.sol#L2

```solidity
File: /contracts/Pool/PoolRegistryInterface.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistryInterface.sol#L2

```solidity
File: /contracts/MaxLoopsLimitHelper.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L2

```solidity
File: /contracts/InterestRateModel.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol#L2

```solidity
File: /contracts/RiskFund/IRiskFund.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IRiskFund.sol#L2

```solidity
File: /contracts/IPancakeswapV2Router.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/IPancakeswapV2Router.sol#L2

```solidity
File: /contracts/Factories/WhitePaperInterestRateModelFactory.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/WhitePaperInterestRateModelFactory.sol#L2

```solidity
File: /contracts/Proxy/UpgradeableBeacon.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-s/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L2

```solidity
File: /contracts/RiskFund/IProtocolShareReserve.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IProtocolShareReserve.sol#L2

```solidity
File: /contracts/Shortfall/IShortfall.sol

2    pragma solidity 0.8.13;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/IShortfall.sol#L2

## [G-03]  Massive 15k per tx gas savings - use 1 and 2 for `Reentrancy guard`

### Summary

Using true and false will trigger gas-refunds, which after London are 1/5 of what they used to be, meaning using 1 and 2 (keeping the slot non-zero), will cost 5k per change (5k + 5k) vs 20k + 5k, saving you 15k gas per function which uses the modifier.so this modifier use in 20 functions

see solmate implementation
https://github.com/transmissions11/solmate/blob/main/src/utils/ReentrancyGuard.sol

### Github Permalinks

```solidity
File: /contracts/VToken.sol

33    modifier nonReentrant() {
34        require(_notEntered, "re-entered");
35        _notEntered = false;
36        _;
37        _notEntered = true; // get a gas-refund post-Istanbul
38    }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L33-L38

## [G-04]  `<X> += <Y>` costes more gas than`<X> = <X> + <Y>` for state variables

### Summary

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

66            assetsReserves[asset] += balanceDifference;

67            poolsAssetsReserves[comptroller][asset] += balanceDifference;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol

### Recommendation Code

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

-66            assetsReserves[asset] += balanceDifference;

+66            assetsReserves[asset] = assetsReserves[asset] +  balanceDifference;
```

## [G-05]  Not using the named return variables when a function `returns` , wastes deployment gas

### Details

There are **10** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

988        returns (
            uint256 error,
            uint256 liquidity,
            uint256 shortfall
        )

1017        returns (
            uint256 error,
            uint256 liquidity,
            uint256 shortfall
        )

1088      returns (uint256 error, uint256 tokensToSeize) {

1119      returns (RewardSpeeds[] memory rewardSpeeds) {

1279       returns (AccountLiquiditySnapshot memory snapshot)

1302        returns (AccountLiquiditySnapshot memory snapshot) {

1405        returns (
            uint256 vTokenBalance,
            uint256 borrowBalance,
            uint256 exchangeRateMantissa
        )
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

565      returns (
            uint256 error,
            uint256 vTokenBalance,
            uint256 borrowBalance,
            uint256 exchangeRate
        )
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L565

```solidity
File: /contracts/Pool/PoolRegistry.sol

222      returns (uint256 index, address proxyAddress) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L222

```solidity
File: /contracts/IPancakeswapV2Router.sol

11    ) external returns (uint256[] memory amounts);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/IPancakeswapV2Router.sol#L11

## [G-06]  Can make the `variable` outside the loop to save gas

### Summary

Consider making the stack variables before the loop which gonna save gas

### Details

There are **18** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

163            VToken vToken = VToken(vTokens[i]);

403            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });

612            VToken market = userAssets[i];

933            address rewardToken = address(rewardsDistributors[i].rewardToken());

1123            address rewardToken = address(rewardsDistributors[i].rewardToken());

1308            VToken asset = assets[i];
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/Lens/PoolLens.sol

431            uint256 borrowReward = calculateBorrowerReward(

438            uint256 supplyReward = calculateSupplierReward(
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

```solidity
File: /contracts/Shortfall/Shortfall.sol

176            VToken vToken = VToken(address(auction.markets[i]));

177            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

224            VToken vToken = VToken(address(auction.markets[i]));

225            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

375            VToken vToken = auction.markets[i];

390            uint256 marketBadDebt = vTokens[i].badDebt();

393            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

```solidity
File: /contracts/Pool/PoolRegistry.sol

357            address comptroller = _poolsByID[i];
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L357

```solidity
File: /contracts/RiskFund/RiskFund.sol

168            address comptroller = address(vToken.comptroller());

174            uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

### Recommendation Code

```solidity
File:  /contracts/RiskFund/RiskFund.sol

+           address comptroller;

166     for (uint256 i; i < marketsCount; ++i) {
                VToken vToken = VToken(markets[i]);
-               address comptroller = address(vToken.comptroller());
+               comptroller = address(vToken.comptroller());

                PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
                require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
                require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");

                uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
                poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;
                totalAmount = totalAmount + swappedTokens;
            }
```

## [G-07]  Using `calldata` instead of `memory` for read-only arguments in external functions save gas

### Summary

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved.

### Details

There are **6** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

/// @audit address[] memory vTokens
154    function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/Lens/PoolLens.sol

388    function vTokenMetadataAll(VToken[] memory vTokens) public view returns (VTokenMetadata[] memory) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

198      VToken[] memory vTokens,

199        uint256[] memory supplySpeeds,

200        uint256[] memory borrowSpeeds

277    function claimRewardToken(address holder, VToken[] memory vTokens) public {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

## [G-08]  Use `assembly` to check for `address(0)`

### Details

There are **45** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

128        require(poolRegistry_ != address(0), "invalid pool registry address");

962        require(address(newOracle) != address(0), "invalid price oracle address");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

72        require(admin_ != address(0), "invalid admin address");

134        require(spender != address(0), "invalid spender address");

196        require(minter != address(0), "invalid minter address");

626        require(spender != address(0), "invalid spender address");

646        require(spender != address(0), "invalid spender address");

1399        if (shortfall_ == address(0)) {

1408        if (protocolShareReserve_ == address(0)) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

```solidity
File: /contracts/Shortfall/Shortfall.sol

137        require(convertibleBaseAsset_ != address(0), "invalid base asset address");

138        require(address(riskFund_) != address(0), "invalid risk fund address");

166                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||

167                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||

169                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||

170                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),

180                if (auction.highestBidder != address(0)) {

189                if (auction.highestBidder != address(0)) {

214            block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),

349        require(_poolRegistry != address(0), "invalid address");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

```solidity
File: /contracts/Pool/PoolRegistry.sol

225        require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

226        require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

257        require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

258        require(input.asset != address(0), "PoolRegistry: Invalid asset address");

259        require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

260        require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

264            _vTokens[input.comptroller][input.asset] == address(0),

396        require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");

422        if (address(shortfall_) == address(0)) {

431        if (protocolShareReserve_ == address(0)) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

```solidity
File: /contracts/RiskFund/RiskFund.sol

80        require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

81        require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

100        require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

111        require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");

127        require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

157        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

```solidity
File: /contracts/BaseJumpRateModelV2.sol

72        require(address(accessControlManager_) != address(0), "invalid ACM address");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L72

```solidity
File: /contracts/RiskFund/ProtocolShareReserve.sol

40        require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

41        require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

54        require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

71        require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

40        require(asset != address(0), "ReserveHelpers: Asset address invalid");

52        require(asset != address(0), "ReserveHelpers: Asset address invalid");

53        require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");

55            PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol

```solidity
File: /contracts/Proxy/UpgradeableBeacon.sol

8        require(implementation_ != address(0), "Invalid implementation address");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L8

## [G-09] State variables only set in the constructor should be declared `immutable`

### Summary

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/BaseJumpRateModelV2.sol

74        accessControlManager = accessControlManager_;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L74

## [G-10]  Use in constants instead of `type(uintx).max`

### Summary

type(uint256).max it uses more gas in the distribution process and also for each transaction than constant usage.

### Details

There are **5** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

262        if (supplyCap != type(uint256).max) {

351        if (borrowCap != type(uint256).max) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

1055        if (repayAmount == type(uint256).max) {

1314            startingAllowance = type(uint256).max;

1331        if (startingAllowance != type(uint256).max) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

## [G-11]  Duplicated `if()` checks should be refactored to a modifier or function

### Details

There are **11** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

256        if (!markets[vToken].isListed) {
333        if (!markets[vToken].isListed) {
395        if (!markets[vToken].isListed) {
1245        if (!markets[vToken].isListed) {


367        if (snapshot.shortfall > 0) {
1262        if (snapshot.shortfall > 0) {


442        if (!markets[vTokenCollateral].isListed) {
497        if (!markets[vTokenCollateral].isListed) {


464        if (snapshot.shortfall == 0) {
595        if (snapshot.shortfall == 0) {
661        if (snapshot.shortfall == 0) {


591        if (snapshot.totalCollateral > minLiquidatableCollateral) {
646        if (snapshot.totalCollateral > minLiquidatableCollateral) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

395        if (msg.sender != address(comptroller)) {
456        if (msg.sender != address(comptroller)) {


752        if (accrualBlockNumber != _getBlockNumber()) {
808        if (accrualBlockNumber != _getBlockNumber()) {
882        if (accrualBlockNumber != _getBlockNumber()) {
939        if (accrualBlockNumber != _getBlockNumber()) {
1035        if (accrualBlockNumber != _getBlockNumber()) {
1156        if (accrualBlockNumber != _getBlockNumber()) {
1183        if (accrualBlockNumber != _getBlockNumber()) {
1205        if (accrualBlockNumber != _getBlockNumber()) {
1248        if (accrualBlockNumber != _getBlockNumber()) {


1045        if (borrower == liquidator) {
1107        if (borrower == liquidator) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

```solidity
File: /contracts/Shortfall/Shortfall.sol

179            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
227            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
241            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {


180                if (auction.highestBidder != address(0)) {
189                if (auction.highestBidder != address(0)) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

```solidity
File: /Rewards/RewardsDistributor.sol

446        } else if (deltaBlocks > 0) {
474        } else if (deltaBlocks > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L446

## [G-12]  Use `nested if` and, avoid multiple check `combinations` &&

### Details

There are **12** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

755        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755

```solidity
File: /contracts/VToken.sol

837        if (redeemTokens == 0 && redeemAmount > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837

```solidity
File: /contracts/Lens/PoolLens.sol

462        if (deltaBlocks > 0 && borrowSpeed > 0) {

483        if (deltaBlocks > 0 && supplySpeed > 0) {

506        if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

526        if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

261        if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

348        if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {

386        if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {

418        if (amount > 0 && amount <= rewardTokenRemaining) {

435        if (deltaBlocks > 0 && supplySpeed > 0) {

463        if (deltaBlocks > 0 && borrowSpeed > 0) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

### Recommendation Code

```solidity
File: /contracts/VToken.sol

-837        if (redeemTokens == 0 && redeemAmount > 0) {

+           if (redeemTokens == 0 ) {
                if(redeemAmount > 0){

                }
    }
```

## [G-13] Use `assembly` to write address storage values 

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

130        poolRegistry = poolRegistry_;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L130

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

117        comptroller = comptroller_;

118        rewardToken = rewardToken_;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

```solidity
File: /contracts/Pool/PoolRegistry.sol

435        protocolShareReserve = protocolShareReserve_;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L435

```solidity
File: /contracts/BaseJumpRateModelV2.sol

74        accessControlManager = accessControlManager_;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L74

```solidity
File: /contracts/RiskFund/ProtocolShareReserve.sol

45        protocolIncome = _protocolIncome;

46        riskFund = _riskFund;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol

### Recommendation Code

```solidity
File: /ajna-core/src/RewardsManager.sol

127     constructor(address poolRegistry_) {
128            require(poolRegistry_ != address(0), "invalid pool registry address");
129
-130            poolRegistry = poolRegistry_;

+           assembly {
                sstore(poolRegistry.slot, poolReistry)
                }
```

## [G-14] Do not calculate `constants` 

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/ExponentialNoError.sol

22    uint256 internal constant halfExpScale = expScale / 2;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22

## [G-15]  Use hardcode address instead `address(this)`

### Summary

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

### Details

There are **30** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Comptroller.sol

/// @audit address(this )
501        if (seizerContract == address(this)) {

504            if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
File: /contracts/VToken.sol

420            emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);

527        uint256 balance = token.balanceOf(address(this));

749        comptroller.preMintHook(address(this), minter, mintAmount);

842        comptroller.preRedeemHook(address(this), redeemer, redeemTokens);

869        emit Transfer(redeemer, address(this), redeemTokens);

879        comptroller.preBorrowHook(address(this), borrower, borrowAmount);

936        comptroller.preRepayHook(address(this), borrower);

1027            address(this),

1068            address(this),

1078        if (address(vTokenCollateral) == address(this)) {

1079            _seize(address(this), liquidator, borrower, seizeTokens);

1104        comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);

1134        emit Transfer(borrower, address(this), protocolSeizeTokens);

1135        emit ReservesAdded(address(this), protocolSeizeAmount, totalReservesNew);

1274        uint256 balanceBefore = token.balanceOf(address(this));

1275        token.safeTransferFrom(from, address(this), amount);

1276        uint256 balanceAfter = token.balanceOf(address(this));

1304        comptroller.preTransferHook(address(this), src, dst, tokens);

1423        return token.balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

```solidity
File: /contracts/Shortfall/Shortfall.sol

187                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);

193                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

417        uint256 rewardTokenRemaining = rewardToken.balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L417

```solidity
File: /contracts/Pool/PoolRegistry.sol

415        uint256 balanceBefore = token.balanceOf(address(this));

416        token.safeTransferFrom(from, address(this), amount);

417        uint256 balanceAfter = token.balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

```solidity
File: /contracts/RiskFund/RiskFund.sol

264                        address(this),
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L264

```solidity
File: /contracts/BaseJumpRateModelV2.sol

98            revert Unauthorized(msg.sender, address(this), signature);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L98

```solidity
File: /contracts/RiskFund/ReserveHelpers.sol

59        uint256 currentBalance = IERC20Upgradeable(asset).balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L59

## [G-16] Use `!= 0` instead of `> 0` for unsigned integer comparison to save gas

### Details

There are **6** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/VToken.sol

818        if (redeemTokensIn > 0) {

837        if (redeemTokens == 0 && redeemAmount > 0) {

1369        require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

```solidity
File: /contracts/RiskFund/RiskFund.sol

82        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

83        require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

139        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

## [G-17] Use `uint256(1)/uint256(2)` instead for `true` and `false` boolean states

### Summary

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/VTokenInterfaces.sol

142    bool public constant isVToken = true;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L142

```solidity
File: /contracts/ComptrollerStorage.sol

115    bool internal constant _isComptroller = true;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L115

```solidity
File: /contracts/InterestRateModel.sol

10    bool public constant isInterestRateModel = true;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol#L10

## [G-18]  Shorten the `array` rather than copying to a `new` one

### Summary

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Pool/PoolRegistry.sol

306        uint256[] memory newSupplyCaps = new uint256[](1);

307        uint256[] memory newBorrowCaps = new uint256[](1);

308        uint256[] memory newBorrowCaps = new uint256[](1);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

## [G-19]  Usage of `uints/ints` smaller than `32 BYTES` 256bits incurs overhead

### Summary

### Details

There are **7** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/VToken.sol

66        uint8 decimals_,

1357        uint8 decimals_,
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

```solidity
File: /contracts/Rewards/RewardsDistributor.sol

29    uint224 public constant rewardTokenInitialIndex = 1e36;

126        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

433        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

461        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

```solidity
File: /contracts/VTokenInterfaces.sol

45    uint8 public decimals;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L45
