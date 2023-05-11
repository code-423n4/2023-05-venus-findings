## Gas Optimizations

|        | Issue                                                                                                 |
| ------ | :---------------------------------------------------------------------------------------------------- |
| GAS-1  | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables |
| GAS-2  | Use assembly to check for `address(0)`                                                                |
| GAS-3  | `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants) |
| GAS-4  | Use calldata instead of memory for function arguments that do not get mutated                         |
| GAS-5  | Setting the constructor to payable                                                                    |
| GAS-6  | USE FUNCTION INSTEAD OF MODIFIERS                                                                     |
| GAS-7  | Don't initialize variables with default value                                                         |
| GAS-8  | The increment in for loop postcondition can be made unchecked                                         |
| GAS-9  | USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION                                |
| GAS-10 | Use shift Right/Left instead of division/multiplication if possible                                   |
| GAS-11 | State variables only set in the constructor should be declared immutable                              |
| GAS-12 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                   |
| GAS-13 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead                                |
| GAS-14 | Use != 0 instead of > 0 for unsigned integer comparison                                               |
| GAS-15 | USE BYTES32 INSTEAD OF STRING                                                                         |
| GAS-16 | USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT                       |

### [GAS-1] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

#### Description:

Using the addition/substraction operator instead of plus-equals/minus-equals saves gas

#### **Proof Of Concept**

```solidity
File: RiskFund/ProtocolShareReserve.sol

74:         assetsReserves[asset] -= amount;

75:         poolsAssetsReserves[comptroller][asset] -= amount;

```

```solidity
File: RiskFund/ReserveHelpers.sol

66:             assetsReserves[asset] += balanceDifference;

67:             poolsAssetsReserves[comptroller][asset] += balanceDifference;

```

```solidity
File: RiskFund/RiskFund.sol

249:                 assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;

250:                 poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;

```

```solidity
File: VToken.sol

630:         newAllowance += addedValue;

652:             currentAllowance -= subtractedValue;

```

### [GAS-2] Use assembly to check for `address(0)`

#### Description:

_Saves 6 gas per instance_

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

72:         require(address(accessControlManager_) != address(0), "invalid ACM address");

```

```solidity
File: Comptroller.sol

128:         require(poolRegistry_ != address(0), "invalid pool registry address");

962:         require(address(newOracle) != address(0), "invalid price oracle address");

```

```solidity
File: Pool/PoolRegistry.sol

225:         require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

226:         require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

257:         require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

258:         require(input.asset != address(0), "PoolRegistry: Invalid asset address");

259:         require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

260:         require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

264:             _vTokens[input.comptroller][input.asset] == address(0),

396:         require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");

422:         if (address(shortfall_) == address(0)) {

431:         if (protocolShareReserve_ == address(0)) {

```

```solidity
File: Proxy/UpgradeableBeacon.sol

8:         require(implementation_ != address(0), "Invalid implementation address");

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

40:         require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

41:         require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

54:         require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

71:         require(asset != address(0), "ProtocolShareReserve: Asset address invalid");

```

```solidity
File: RiskFund/ReserveHelpers.sol

40:         require(asset != address(0), "ReserveHelpers: Asset address invalid");

52:         require(asset != address(0), "ReserveHelpers: Asset address invalid");

53:         require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");

55:             PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),

```

```solidity
File: RiskFund/RiskFund.sol

80:         require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

81:         require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

100:         require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

111:         require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");

127:         require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

157:         require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");

```

```solidity
File: Shortfall/Shortfall.sol

137:         require(convertibleBaseAsset_ != address(0), "invalid base asset address");

138:         require(address(riskFund_) != address(0), "invalid risk fund address");

166:                 ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||

167:                     (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||

169:                     ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||

170:                         (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),

180:                 if (auction.highestBidder != address(0)) {

189:                 if (auction.highestBidder != address(0)) {

214:             block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),

349:         require(_poolRegistry != address(0), "invalid address");

468:         bool noBidder = auction.highestBidder == address(0);

```

```solidity
File: VToken.sol

72:         require(admin_ != address(0), "invalid admin address");

134:         require(spender != address(0), "invalid spender address");

196:         require(minter != address(0), "invalid minter address");

626:         require(spender != address(0), "invalid spender address");

646:         require(spender != address(0), "invalid spender address");

1399:         if (shortfall_ == address(0)) {

1408:         if (protocolShareReserve_ == address(0)) {

```

### [GAS-3] `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants)

#### Description:

When updating a value in an array with arithmetic, using `array[index] += amount` is cheaper than `array[index] = array[index] + amount`.
This is because you avoid an additonal `mload` when the array is stored in memory, and an `sload` when the array is stored in storage.
This can be applied for any arithmetic operation including `+=`, `-=`,`/=`,`*=`,`^=`,`&=`, `%=`, `<<=`,`>>=`, and `>>>=`.
This optimization can be particularly significant if the pattern occurs during a loop.

_Saves 28 gas for a storage array, 38 for a memory array_
[Source](https://dev.to/juanxavier/advanced-gas-optimizations-tips-for-solidity-1j2f)

#### **Proof Of Concept**

```solidity
File: RiskFund/RiskFund.sol

175:             poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;

193:         poolReserves[comptroller] = poolReserves[comptroller] - amount;

```

```solidity
File: VToken.sol

1129:         accountTokens[borrower] = accountTokens[borrower] - seizeTokens;

1130:         accountTokens[liquidator] = accountTokens[liquidator] + liquidatorSeizeTokens;

```

### [GAS-4] Use calldata instead of memory for function arguments that do not get mutated

#### Description:

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)
[Source 2](https://dev.to/juanxavier/advanced-gas-optimizations-tips-for-solidity-1j2f)

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

154:     function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {

```

```solidity
File: Factories/VTokenProxyFactory.sol

28:     function deployVTokenProxy(VTokenArgs memory input) external returns (VToken) {

```

```solidity
File: Lens/PoolLens.sol

309:     function getPoolDataFromVenusPool(address poolRegistryAddress, PoolRegistry.VenusPool memory venusPool)

388:     function vTokenMetadataAll(VToken[] memory vTokens) public view returns (VTokenMetadata[] memory) {

```

```solidity
File: Pool/PoolRegistry.sol

255:     function addMarket(AddMarketInput memory input) external {

343:     function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {

```

```solidity
File: Rewards/RewardsDistributor.sol

166:         Exp memory marketBorrowIndex

187:     function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {

198:         VToken[] memory vTokens,

199:         uint256[] memory supplySpeeds,

200:         uint256[] memory borrowSpeeds

277:     function claimRewardToken(address holder, VToken[] memory vTokens) public {

```

```solidity
File: VToken.sol

64:         string memory name_,

65:         string memory symbol_,

69:         RiskManagementInit memory riskManagement,

```

### [GAS-5] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

65:     constructor(

```

```solidity
File: Comptroller.sol

127:     constructor(address poolRegistry_) {

```

```solidity
File: JumpRateModelV2.sol

12:     constructor(

```

```solidity
File: Pool/PoolRegistry.sol

150:     constructor() {

```

```solidity
File: Proxy/UpgradeableBeacon.sol

7:     constructor(address implementation_) UpgradeableBeacon(implementation_) {

```

```solidity
File: Rewards/RewardsDistributor.sol

104:     constructor() {

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

28:     constructor() {

```

```solidity
File: RiskFund/RiskFund.sol

61:     constructor() {

```

```solidity
File: Shortfall/Shortfall.sol

118:     constructor() {

```

```solidity
File: VToken.sol

41:     constructor() {

```

```solidity
File: WhitePaperInterestRateModel.sol

36:     constructor(uint256 baseRatePerYear, uint256 multiplierPerYear) {

```

### [GAS-6] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: Rewards/RewardsDistributor.sol

98:     modifier onlyComptroller() {

```

```solidity
File: VToken.sol

33:     modifier nonReentrant() {

```

### [GAS-7] Don't initialize variables with default value

#### Description:

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

#### **Proof Of Concept**

```solidity
File: ComptrollerStorage.sol

103:     uint256 internal constant NO_ERROR = 0;

```

```solidity
File: ErrorReporter.sol

5:     uint256 public constant NO_ERROR = 0; // support legacy return codes

```

### [GAS-8] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: Pool/PoolRegistry.sol

399:         _numberOfPools++;

```

### [GAS-9] USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Comptroller.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ComptrollerInterface.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ComptrollerStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ErrorReporter.sol

2: pragma solidity 0.8.13;

```

```solidity
File: ExponentialNoError.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/JumpRateModelFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/VTokenProxyFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Factories/WhitePaperInterestRateModelFactory.sol

2: pragma solidity 0.8.13;

```

```solidity
File: IPancakeswapV2Router.sol

2: pragma solidity 0.8.13;

```

```solidity
File: InterestRateModel.sol

2: pragma solidity 0.8.13;

```

```solidity
File: JumpRateModelV2.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Lens/PoolLens.sol

2: pragma solidity 0.8.13;

```

```solidity
File: MaxLoopsLimitHelper.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Pool/PoolRegistry.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Pool/PoolRegistryInterface.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Proxy/UpgradeableBeacon.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Rewards/RewardsDistributor.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/IProtocolShareReserve.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/IRiskFund.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/ProtocolShareReserve.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/ReserveHelpers.sol

2: pragma solidity 0.8.13;

```

```solidity
File: RiskFund/RiskFund.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Shortfall/IShortfall.sol

2: pragma solidity 0.8.13;

```

```solidity
File: Shortfall/Shortfall.sol

2: pragma solidity 0.8.13;

```

```solidity
File: VToken.sol

2: pragma solidity 0.8.13;

```

```solidity
File: VTokenInterfaces.sol

2: pragma solidity 0.8.13;

```

```solidity
File: WhitePaperInterestRateModel.sol

2: pragma solidity 0.8.13;

```

### [GAS-10] Use shift Right/Left instead of division/multiplication if possible

#### **Proof Of Concept**

```solidity
File: ExponentialNoError.sol

22:     uint256 internal constant halfExpScale = expScale / 2;

```

### [GAS-11] State variables only set in the constructor should be declared immutable

#### Description:

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

65:     constructor(

```

### [GAS-12] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: VToken.sol

1313         if (spender == src) {
1314             startingAllowance = type(uint256).max;
1315         } else {
1316             startingAllowance = transferAllowances[src][spender];
1317:        }

```

### [GAS-13] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: Factories/VTokenProxyFactory.sol

18:         uint8 decimals_;

```

```solidity
File: Lens/PoolLens.sol

99:         uint32 block;

```

```solidity
File: Pool/PoolRegistry.sol

36:         uint8 decimals;

```

```solidity
File: Rewards/RewardsDistributor.sol

19:         uint32 block;

126:         uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

433:         uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

461:         uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

```

```solidity
File: VToken.sol

66:         uint8 decimals_,

1357:         uint8 decimals_,

```

```solidity
File: VTokenInterfaces.sol

45:     uint8 public decimals;

```

### [GAS-14] Use != 0 instead of > 0 for unsigned integer comparison

#### Description:

To update this with using at least 0.8.6 there is no difference in gas usage with!= 0 or > 0.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: Comptroller.sol

367:         if (snapshot.shortfall > 0) {

1262:         if (snapshot.shortfall > 0) {

```

```solidity
File: Lens/PoolLens.sol

462:         if (deltaBlocks > 0 && borrowSpeed > 0) {

462:         if (deltaBlocks > 0 && borrowSpeed > 0) {

466:             Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });

470:         } else if (deltaBlocks > 0) {

483:         if (deltaBlocks > 0 && supplySpeed > 0) {

483:         if (deltaBlocks > 0 && supplySpeed > 0) {

486:             Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });

490:         } else if (deltaBlocks > 0) {

506:         if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

526:         if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {

```

```solidity
File: Rewards/RewardsDistributor.sol

261:         if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

261:         if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

418:         if (amount > 0 && amount <= rewardTokenRemaining) {

435:         if (deltaBlocks > 0 && supplySpeed > 0) {

435:         if (deltaBlocks > 0 && supplySpeed > 0) {

438:             Double memory ratio = supplyTokens > 0

446:         } else if (deltaBlocks > 0) {

463:         if (deltaBlocks > 0 && borrowSpeed > 0) {

463:         if (deltaBlocks > 0 && borrowSpeed > 0) {

466:             Double memory ratio = borrowAmount > 0

474:         } else if (deltaBlocks > 0) {

```

```solidity
File: RiskFund/RiskFund.sol

82:         require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

83:         require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

139:         require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

244:         if (balanceOfUnderlyingAsset > 0) {

```

```solidity
File: VToken.sol

818:         if (redeemTokensIn > 0) {

837:         if (redeemTokens == 0 && redeemAmount > 0) {

1369:         require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");

```

### [GAS-15] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: BaseJumpRateModelV2.sol

55:     error Unauthorized(address sender, address calledContract, string methodSignature);

94:         string memory signature = "updateJumpRateModel(uint256,uint256,uint256,uint256)";

```

```solidity
File: ExponentialNoError.sol

63:     function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {

68:     function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {

```

```solidity
File: Factories/VTokenProxyFactory.sol

16:         string name_;

17:         string symbol_;

```

```solidity
File: Lens/PoolLens.sol

17:         string name;

22:         string category;

23:         string logoURL;

24:         string description;

```

```solidity
File: Pool/PoolRegistry.sol

37:         string name;

38:         string symbol;

118:     event PoolNameSet(address indexed comptroller, string oldName, string newName);

118:     event PoolNameSet(address indexed comptroller, string oldName, string newName);

214:         string calldata name,

223:         _checkAccessAllowed("createRegistryPool(string,address,uint256,uint256,uint256,address,uint256,address)");

332:     function setPoolName(address comptroller, string calldata name) external {

333:         _checkAccessAllowed("setPoolName(address,string)");

335:         string memory oldName = _poolByComptroller[comptroller].name;

393:     function _registerPool(string calldata name, address comptroller) internal returns (uint256) {

439:     function _ensureValidName(string calldata name) internal pure {

```

```solidity
File: Pool/PoolRegistryInterface.sol

9:         string name;

20:         string category;

21:         string logoURL;

22:         string description;

```

```solidity
File: VToken.sol

64:         string memory name_,

65:         string memory symbol_,

1355:         string memory name_,

1356:         string memory symbol_,

```

```solidity
File: VTokenInterfaces.sol

35:     string public name;

40:     string public symbol;

```

### [GAS-16] USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Description:

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

#### **Proof Of Concept**

```solidity
File: RiskFund/RiskFund.sol

82:         require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

83:         require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

139:         require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

```

```solidity
File: VToken.sol

1369:         require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");

```
