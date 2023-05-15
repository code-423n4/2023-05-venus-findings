|Name|problem|instance
|----|-------|--------
| [G-1] |Change public function visibility to external|10
|[G-2]| bytes constants are more eficient than string constans|2
|[G‑3] |Functions guaranteed to revert when called by normal users can be marked payable|13
|[G-4] | Use hardcode address instead address(this)|32
|[G-5]| Use solidity version 0.8.19 to gain some gas boost|10
|[G-6]| Use double require instead of using &&|3
|[G-7]| Transfer ERC20 immediately to the user|10
|[G-8] |Use double if statements instead of &&|13
 [G-9] |Use calldata instead of memory|1
|[G-10]| State variables can be packed to use fewer storage slots|2
|[G-11] |Use constants instead of type(uintx).max|5
|[G-12]| Remove the initializer modifier|6
|[G-13]|Not using the named return variables when a function returns, wastes deployment gas|2
|[G-14] |Avoid using state variable in emit |3








### [G-1] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization
...solidity
    352 function vTokenMetadata(VToken vToken) public view returns (VTokenMetadata memory) {
    310 public
        view
        returns (PoolData memory)
    352   function vTokenMetadata(VToken vToken) public view returns (VTokenMetadata memory) {

    388   function vTokenMetadataAll(VToken[] memory vTokens) public view returns (VTokenMetadata[] memory) {
    401   function vTokenUnderlyingPrice(VToken vToken) public view returns (VTokenUnderlyingPrice memory) {
          
    1144  function getRewardDistributors() public view returns (RewardsDistributor[] memory) 
    1154  function actionPaused(address market, Action action) public view returns (bool)
 
    1164  function isDeprecated(VToken vToken) public view returns (bool)
    54   public view override returns (uint256)
    72   public view override returns (uint256)
    ...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L352
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L310
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L352
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L388
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L401
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1144
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1154
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1164
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L54
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L72

### [G-2] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.
...solidity
 35  string public name;
 40  string public symbol;
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L35
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L40

### [G‑3] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
...solidity
   
   927 function addRewardsDistributor(RewardsDistributor _rewardsDistributor) external onlyOwner 
   961  function setPriceOracle(PriceOracle newOracle) external onlyOwner 
   973  function setMaxLoopsLimit(uint256 limit) external onlyOwner 
   515  function setShortfallContract(address shortfall_) external onlyOwner
   219    function setContributorRewardTokenSpeed(address contributor, uint256 rewardTokenSpeed) external      onlyOwner 
   249  function setMaxLoopsLimit(uint256 limit) external onlyOwner
   198     function setShortfallContract(Shortfall shortfall_) external onlyOwner 
   99  function setPoolRegistry(address _poolRegistry) external onlyOwner
   110  function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner
   126  function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner
   205     function setMaxLoopsLimit(uint256 limit) external onlyOwner
   53  function setPoolRegistry(address _poolRegistry) external onlyOwner
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L927
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L973
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L515
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L219
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L249
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L198
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L110
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L205
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L53

### [G-04] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

...solidity
501  if (seizerContract == address(this))
504   if (address(VToken(vTokenCollateral).comptroller()) != address(this))
420   emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);
527    uint256 balance = token.balanceOf(address(this));
749   comptroller.preMintHook(address(this), minter, mintAmount);
842    comptroller.preRedeemHook(address(this), redeemer, redeemTokens);
869    emit Transfer(redeemer, address(this), redeemTokens);
879    comptroller.preBorrowHook(address(this), borrower, borrowAmount);
936    comptroller.preRepayHook(address(this), borrower);
1027    comptroller.preLiquidateHook
        address(this),
1068     (uint256 amountSeizeError, uint256 seizeTokens) = comptroller.liquidateCalculateSeizeTokens
            address(this),
1078     if (address(vTokenCollateral) == address(this))
1079         _seize(address(this), liquidator, borrower, seizeTokens);
1104       comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);
1134     emit Transfer(borrower, address(this), protocolSeizeTokens);
1135      emit ReservesAdded(address(this), protocolSeizeAmount, totalReservesNew);
1174   uint256 balanceBefore = token.balanceOf(address(this));
1175     token.safeTransferFrom(from, address(this), amount);
 1176       uint256 balanceAfter = token.balanceOf(address(this));
 1304      comptroller.preTransferHook(address(this), src, dst, tokens);
 1423    return token.balanceOf(address(this));
 187   erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
 193   erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
 417  uint256 rewardTokenRemaining = rewardToken.balanceOf(address(this));
 415  uint256 balanceBefore = token.balanceOf(address(this));
416      token.safeTransferFrom(from, address(this), amount);
417      uint256 balanceAfter = token.balanceOf(address(this));
264     address(this),
59   uint256 currentBalance = IERC20Upgradeable(asset).balanceOf(address(this));
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L501
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L504
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L420
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L527
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L749
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L842
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L869
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L879
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L936
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1027
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1068
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1078
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1079
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1104
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1134
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1135
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1274
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1275
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1276
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1304
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1423
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L187
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L193
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L417
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L415
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L416
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L417
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L264
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L59

### [G-5] Use solidity version 0.8.19 to gain some gas boost
Upgrade to the latest solidity version 0.8.19 to get additional gas savings.

See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

...solidity
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
2 pragma solidity 0.8.13;
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L2
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L2

### [G-6] Use double require instead of using &&
Using double require instead of operator && can save more gas.
When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.
...solidity

214 require(
            block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),
365   require(
            (auction.startBlock == 0 && auction.status == AuctionStatus.NOT_STARTED) || 
205  require(
            numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,            
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L214
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L365
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L205

### [G-07] Transfer ERC20 immediately to the user
...solidity
26  using SafeERC20Upgradeable for IERC20Upgradeable;
290  IERC20 underlying = IERC20(vToken.underlying());
18 using SafeERC20Upgradeable for IERC20Upgradeable;
177  IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
255    IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
13    using SafeERC20Upgradeable for IERC20Upgradeable;
26  using SafeERC20Upgradeable for IERC20Upgradeable;
27  using SafeERC20Upgradeable for IERC20Upgradeable;
13  using SafeERC20Upgradeable for IERC20Upgradeable;
10  using SafeERC20Upgradeable for IERC20Upgradeable;
... 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L26
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L290
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L18
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L177
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L225
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L13
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L26
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L27
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L13
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L10

### [G-08] Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

...solidity
755  if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0)
842         require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");
837 if (redeemTokens == 0 && redeemAmount > 0)
462     if (deltaBlocks > 0 && borrowSpeed > 0)
483  if (deltaBlocks > 0 && supplySpeed > 0)
506  if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0)
526 if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0)
261   if (deltaBlocks > 0 && rewardTokenSpeed > 0)
348 if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex)

386 if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex)
418  if (amount > 0 && amount <= rewardTokenRemaining)
435   if (deltaBlocks > 0 && supplySpeed > 0)
463    if (deltaBlocks > 0 && borrowSpeed > 0)
...
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L842
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L483
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L348
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L386
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L418
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L435
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L463

### [G-9] Use calldata instead of memory

```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
198   function setRewardTokenSpeeds(
        VToken[] memory vTokens,
        uint256[] memory supplySpeeds,
        uint256[] memory borrowSpeeds
    ) external {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L198-L201

### [G-10] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).
```solidity
File: /main/contracts/VTokenInterfaces.sol
123   address public shortfall;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L123

### [G-11] Use constants instead of type(uintx).max
```solidity
File: /main/contracts/Comptroller.sol
262    if (supplyCap != type(uint256).max) {

351    if (borrowCap != type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L262

```solidity
File: /main/contracts/VToken.sol
1055   if (repayAmount == type(uint256).max) {

1314   startingAllowance = type(uint256).max;

1331   if (startingAllowance != type(uint256).max) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1055
### [G-12] Remove the initializer modifier
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L138

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

### [G-13]Not using the named return variables when a function returns, wastes deployment gas

```solidity
File: /main/contracts/Comptroller.sol
169   return results;

1039  return allMarkets;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L169

### [G-14] Avoid using state variable in emit 


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