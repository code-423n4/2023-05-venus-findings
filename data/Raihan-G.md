# Gas Optimization
|      | Issue   | Instence|
|------|---------|---------|
|[G-01]|Use calldata instead of memory|2|
|[G-02]|Can Make The Variable Outside The Loop To Save Gas |13|
|[G-03]|Use assembly to write address storage values|4|
|[G-04]|Use double if statements instead of &&|8|
|[G-05]|Make 3 event parameters indexed when possible|52|
|[G-06]|Sort Solidity operations using short-circuit mode|1|
|[G-07]|public functions to external|7|
|[G-08]|Amounts should be checked for 0 before calling a transfer|3|
|[G-09]|Do not calculate constants|1|
|[G-10]|Do not shrink Variables|1|
|[G-11]|State variables can be packed to use fewer storage slots|1|
|[G-12]|Use != 0 instead of > 0 for unsigned integer comparison|12|
|[G-13]|Use hardcode address instead address(this)|25|
|[G-14]|Use constants instead of type(uintx).max|5|
|[G-15]| Remove the initializer modifier|7|
|[G-16]|Use assembly for math (add, sub, mul, div)|9|
|[G-17]| Duplicated require()/if() checks should be refactored to a modifier or function|12|
|[G-18]|Not using the named return variables when a function returns, wastes deployment gas|6|
|[G-19]|Unnecessary computation|16|
|[G-20]|bytes constants are more eficient than string constans|1|
|[G-21]|Avoid using state variable in emit |3|
|[G-22]|Gas savings can be achieved by changing the model for assigning value to the structure|3|
|[G-23]|Use Assembly To Check For address(0)|38|
|[G-24]|use Mappings Instead of Arrays|3|
|[G-25]|Using bools for storage incurs overhead|3|

## [G-01] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.

When a function is marked as external, its arguments are passed in the calldata section of the transaction, which is a read-only area of memory that contains the input data for the transaction. Using calldata instead of memory for function arguments can be more gas-efficient, especially for functions that take large arguments.

```solidity
File: /main/contracts/Comptroller.sol
154   function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154

```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
198   function setRewardTokenSpeeds(
        VToken[] memory vTokens,
        uint256[] memory supplySpeeds,
        uint256[] memory borrowSpeeds
    ) external {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L198-L201

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: /main/contracts/Comptroller.sol
614   (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);

615   uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);  

691   (, uint256 borrowBalance, ) = _safeGetAccountSnapshot(borrowMarkets[i], borrower);

933   address rewardToken = address(rewardsDistributors[i].rewardToken());

1123  address rewardToken = address(rewardsDistributors[i].rewardToken());

1311  (uint256 vTokenBalance, uint256 borrowBalance, uint256 exchangeRateMantissa) = _safeGetAccountSnapshot(
                asset,
                account
            )
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L614

```solidity
File: /main/contracts/Lens/PoolLens.sol
431   uint256 borrowReward = calculateBorrowerReward(

438   uint256 supplyReward = calculateSupplierReward(
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L431

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
356   address comptroller = _poolsByID[i];
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L356

```solidity
File: /main/contracts/RiskFund/RiskFund.sol

168   address comptroller = address(vToken.comptroller());

174   uint256 swappedTokens = _swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L168

```solidity
File: /main/contracts/Shortfall/Shortfall.sol
390   uint256 marketBadDebt = vTokens[i].badDebt();

393    uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L390

## [G-03] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```
```solidity
File: /main/contracts/Comptroller.sol
130    poolRegistry = poolRegistry_;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L130


```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
45  protocolIncome = _protocolIncome;

46  riskFund = _riskFund;

56  poolRegistry = _poolRegistry;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L45

## [G-04]Use double if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```

```solidity
File: /main/contracts/Comptroller.sol
755   if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
```    
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755

```solidity
File: /main/contracts/Lens/PoolLens.sol
506    if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

526    if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506

```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
261    if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

348     if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {

386    if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {

418    if (amount > 0 && amount <= rewardTokenRemaining) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261

```solidity
File: /main/contracts/VToken.sol
837   if (redeemTokens == 0 && redeemAmount > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837

## [G-05] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

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
File: /main/contracts/Comptroller.sol
30  event MarketEntered(VToken vToken, address account);

    /// @notice Emitted when an account exits a market
    event MarketExited(VToken vToken, address account);

    /// @notice Emitted when close factor is changed by admin
    event NewCloseFactor(uint256 oldCloseFactorMantissa, uint256 newCloseFactorMantissa);

    /// @notice Emitted when a collateral factor is changed by admin
    event NewCollateralFactor(VToken vToken, uint256 oldCollateralFactorMantissa, uint256 newCollateralFactorMantissa);

    /// @notice Emitted when liquidation threshold is changed by admin
    event NewLiquidationThreshold(
        VToken vToken,
        uint256 oldLiquidationThresholdMantissa,
        uint256 newLiquidationThresholdMantissa
    );

    /// @notice Emitted when liquidation incentive is changed by admin
    event NewLiquidationIncentive(uint256 oldLiquidationIncentiveMantissa, uint256 newLiquidationIncentiveMantissa);

    /// @notice Emitted when price oracle is changed
    event NewPriceOracle(PriceOracle oldPriceOracle, PriceOracle newPriceOracle);

    /// @notice Emitted when an action is paused on a market
    event ActionPausedMarket(VToken vToken, Action action, bool pauseState);

    /// @notice Emitted when borrow cap for a vToken is changed
    event NewBorrowCap(VToken indexed vToken, uint256 newBorrowCap);

    /// @notice Emitted when the collateral threshold (in USD) for non-batch liquidations is changed
    event NewMinLiquidatableCollateral(uint256 oldMinLiquidatableCollateral, uint256 newMinLiquidatableCollateral);

    /// @notice Emitted when supply cap for a vToken is changed
    event NewSupplyCap(VToken indexed vToken, uint256 newSupplyCap);

    /// @notice Emitted when a rewards distributor is added
    event NewRewardsDistributor(address indexed rewardsDistributor);

    /// @notice Emitted when a market is supported
    event MarketSupported(VToken vToken);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L30-L70

```solidity
File: /main/contracts/MaxLoopsLimitHelper.sol
16   event MaxLoopsLimitUpdated(uint256 oldMaxLoopsLimit, uint256 newmaxLoopsLimit);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L16

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
118   event PoolNameSet(address indexed comptroller, string oldName, string newName);

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
File: /main/contracts/Rewards/RewardsDistributor.sol
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
File: /main/contracts/RiskFund/RiskFund.sol

47    event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);

    
50    event MinAmountToConvertUpdated(uint256 oldMinAmountToConvert, uint256 newMinAmountToConvert);

    
    event TransferredReserveForAuction(address comptroller, uint256 amount);

56   event TransferredReserveForAuction(address comptroller, uint256 amount);    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L47


```solidity
File: /main/contracts/Shortfall/Shortfall.sol
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

## [G-06] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

```solidity
File: /main/contracts/Comptroller.sol
449   if (skipLiquidityCheck || isDeprecated(VToken(vTokenBorrowed))) {  
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L449

## [G-07] public functions to external
External call cost is less expensive than of public functions.
Contracts are allowed to override their parents’ functions and change the visibility from external to public.
The following functions could be set external to save gas and improve code quality:

```solidity
File: /main/contracts/BaseJumpRateModelV2.sol
112  function getSupplyRate(
        uint256 cash,
        uint256 borrows,
        uint256 reserves,
        uint256 reserveFactorMantissa
    ) public view virtual override returns (uint256) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L112-117

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
97   function updateAssetsState(address comptroller, address asset)
        public
        override(IProtocolShareReserve, ReserveHelpers)
    {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L97-L103


```solidity
File: /main/contracts/RiskFund/RiskFund.sol
214       function updateAssetsState(address comptroller, address asset) public override(IRiskFund, ReserveHelpers) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L214

```solidity
File: /main/contracts/VToken.sol
665    function exchangeRateCurrent() public override nonReentrant returns (uint256) {
        accrueInterest();
        return _exchangeRateStored();
    }
678    function accrueInterest() public virtual override returns (uint256) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L665

```solidity
File: /main/contracts/WhitePaperInterestRateModel.sol
50    function getBorrowRate(
        uint256 cash,
        uint256 borrows,
        uint256 reserves
    ) public view override returns (uint256) {

72   ) public view override returns (uint256) {
        uint256 oneMinusReserveFactor = BASE - reserveFactorMantissa;
        uint256 borrowRate = getBorrowRate(cash, borrows, reserves);
        uint256 rateToPool = (borrowRate * oneMinusReserveFactor) / BASE;
        return (utilizationRate(cash, borrows, reserves) * rateToPool) / BASE;
    }        
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L50

## [G-08] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
81   IERC20Upgradeable(asset).safeTransfer(protocolIncome, protocolIncomeAmount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L81

```solidity
File: /main/contracts/RiskFund/RiskFund.sol
194    IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L194


```solidity
File: /main/contracts/VToken.sol
1286    token.safeTransfer(to, amount);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1286

## [G-09] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: /main/contracts/ExponentialNoError.sol
22    uint256 internal constant halfExpScale = expScale / 2;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22


##  [G-10]Do not shrink Variables
This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.


```solidity
File: /main/contracts/VTokenInterfaces.sol
45     uint8 public decimals;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L45

## [G-11] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).
```solidity
File: /main/contracts/VTokenInterfaces.sol
123   address public shortfall;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L123

## [G-12]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

```solidity
File: /main/contracts/Lens/PoolLens.sol
462   if (deltaBlocks > 0 && borrowSpeed > 0) {

466   Double memory ratio = borrowAmount > 0 ? fraction(tokensAccrued, borrowAmount) : Double({ mantissa: 0 });

470   } else if (deltaBlocks > 0) {

483   if (deltaBlocks > 0 && supplySpeed > 0) {

486    Double memory ratio = supplyTokens > 0 ? fraction(tokensAccrued, supplyTokens) : Double({ mantissa: 0 });

490    } else if (deltaBlocks > 0) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462


```solidity
File: /main/contracts/Rewards/RewardsDistributor.sol
261    if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

418    if (amount > 0 && amount <= rewardTokenRemaining) {

435    if (deltaBlocks > 0 && supplySpeed > 0) {

438    Double memory ratio = supplyTokens > 0

446    } else if (deltaBlocks > 0) {

463    if (deltaBlocks > 0 && borrowSpeed > 0) { 
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261


## [G-13] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: /main/contracts/BaseJumpRateModelV2.sol
98    revert Unauthorized(msg.sender, address(this), signature);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L98

```solidity
File: /main/contracts/Comptroller.sol
501    if (seizerContract == address(this)) {

504    if (address(VToken(vTokenCollateral).comptroller()) != address(this)) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#501


```solidity
File: /main/contracts/Pool/PoolRegistry.sol
415     uint256 balanceBefore = token.balanceOf(address(this));

416     token.safeTransferFrom(from, address(this)amount);

417     uint256 balanceAfter = token.balanceOf(address(this));
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L415

```solidity
File: /main/contracts/VToken.sol
420    emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);

527     uint256 balance = token.balanceOf(address(this));

749     comptroller.preMintHook(address(this), minter, mintAmount);

842     comptroller.preRedeemHook(address(this), redeemer, redeemTokens);

869     emit Transfer(redeemer, address(this), redeemTokens);

879     comptroller.preBorrowHook(address(this), borrower, borrowAmount);

936     comptroller.preRepayHook(address(this), borrower);

1027    address(this),

1068    address(this),

1078    if (address(vTokenCollateral) == address(this)) {

1079   _seize(address(this), liquidator, borrower, seizeTokens);

1104    comptroller.preSeizeHook(address(this), seizerContract, liquidator, borrower);

1134   emit Transfer(borrower, address(this), protocolSeizeTokens);

1135  emit ReservesAdded(address(this), protocolSeizeAmount, totalReservesNew);

1274   uint256 balanceBefore = token.balanceOf(address(this));

1275   token.safeTransferFrom(from, address(this), amount);

1276   uint256 balanceAfter = token.balanceOf(address(this));

1304   comptroller.preTransferHook(address(this), src, dst, tokens);

1423   return token.balanceOf(address(this));

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L420


## [G-14] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

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

## [G-15] Remove the initializer modifier
If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
File: /main/contracts/Comptroller.sol
138   function initialize(uint256 loopLimit, address accessControlManager) external initializer {

```
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


## [G-16] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```
contract Contract0 {
    //addition in Solidity
    function addTest(uint256 a, uint256 b) public pure {
        uint256 c = a + b;
    }
}
contract Contract1 {
    //addition in assembly
    function addAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)
            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
}
contract Contract2 {
    //subtraction in Solidity
    function subTest(uint256 a, uint256 b) public pure {
        uint256 c = a - b;
    }
}
contract Contract3 {
    //subtraction in assembly
    function subAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := sub(a, b)
            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }
}
contract Contract4 {
    //multiplication in Solidity
    function mulTest(uint256 a, uint256 b) public pure {
        uint256 c = a * b;
    }
}
contract Contract5 {
    //multiplication in assembly
    function mulAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := mul(a, b)
            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
}
contract Contract6 {
    //division in Solidity
    function divTest(uint256 a, uint256 b) public pure {
        uint256 c = a * b;
    }
}
contract Contract7 {
    //division in assembly
    function divAssemblyTest(uint256 a, uint256 b) public pure {
        assembly {
            let c := div(a, b)
            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }
}
```
```solidity
File: /main/contracts/Comptroller.sol
353    uint256 nextTotalBorrows = totalBorrows + borrowAmount;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L353

```solidity
File: /main/contracts/ExponentialNoError.sol
81 function add_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }

95  function sub_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a - b;
    }    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L81

```solidity
File: /main/contracts/VToken.sol
494     uint256 badDebtNew = badDebtOld - recoveredAmount_;

784    totalSupply = totalSupply + mintTokens;

897    uint256 accountBorrowsNew = accountBorrowsPrev + borrowAmount;

898    uint256 totalBorrowsNew = totalBorrows + borrowAmount;

965    uint256 accountBorrowsNew = accountBorrowsPrev - actualRepayAmount;

966    uint256 totalBorrowsNew = totalBorrows - actualRepayAmount;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L494


## [G-17] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```
```solidity
File: /main/contracts/VToken.sol
752   if (accrualBlockNumber != _getBlockNumber()) {

808   if (accrualBlockNumber != _getBlockNumber()) {

882    if (accrualBlockNumber != _getBlockNumber()) {

939    if (accrualBlockNumber != _getBlockNumber()) {

1035   if (accrualBlockNumber != _getBlockNumber()) {

1156  if (accrualBlockNumber != _getBlockNumber()) {

1183  if (accrualBlockNumber != _getBlockNumber()) {

1183  if (accrualBlockNumber != _getBlockNumber()) {

1248  if (accrualBlockNumber != _getBlockNumber()) {   

1045  if (borrower == liquidator) {

1107  if (borrower == liquidator) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L752

## [G-18]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost


```solidity
File: /main/contracts/Comptroller.sol
169   return results;

1039  return allMarkets;

1111  return (NO_ERROR, seizeTokens);

1137  return _isComptroller;

1416  return (vTokenBalance, borrowBalance, exchangeRateMantissa);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L169

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
248    return (poolId, proxyAddress);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L356

## [G-19] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```
```solidity
File: /main/contracts/Comptroller.sol
791   emit NewLiquidationIncentive(oldLiquidationIncentiveMantissa, newLiquidationIncentiveMantissa);

917   emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);

966  emit NewPriceOracle(oldOracle, newOracle);


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L791

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
347   emit PoolMetadataUpdated(comptroller, oldMetadata, _metadata);

436   emit NewProtocolShareReserve(oldProtocolShareReserve, protocolShareReserve_);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L347

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
57   emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L57

```solidity
File: /main/contracts/RiskFund/RiskFund.sol
103   emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);

119   emit ShortfallContractUpdated(oldShortfallContractAddress, shortfallContractAddress_);

130   emit PancakeSwapRouterUpdated(oldPancakeSwapRouter, pancakeSwapRouter_);

142   emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L103


```solidity
File: /main/contracts/Shortfall/Shortfall.sol
298   emit NextBidderBlockLimitUpdated(oldNextBidderBlockLimit, _nextBidderBlockLimit);

312   emit IncentiveBpsUpdated(oldIncentiveBps, _incentiveBps);

325   emit MinimumPoolBadDebtUpdated(oldMinimumPoolBadDebt, _minimumPoolBadDebt);

338   emit WaitForFirstBidderUpdated(oldWaitForFirstBidder, _waitForFirstBidder);

352   emit PoolRegistryUpdated(oldPoolRegistry, _poolRegistry);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L298

```solidity
File: /main/contracts/VToken.sol
319   emit NewProtocolSeizeShare(oldProtocolSeizeShareMantissa, newProtocolSeizeShareMantissa_);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L319

## [G-20] bytes constants are more eficient than string constans
If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
File: /main/contracts/VTokenInterfaces.sol
35   string public name;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L45

## [G-21] Avoid using state variable in emit 
When you use a state variable in an emit statement, Solidity needs to write the value of the state variable to storage in order to include it in the event log. This requires a separate SSTORE operation, which can be quite expensive in terms of gas costs.

In contrast, if you pass a function parameter to an emit statement, Solidity doesn't need to write the value to storage. Instead, it simply includes the value in the event log entry for the transaction.

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


## [G-22] Gas savings can be achieved by changing the model for assigning value to the structure
Here's an example to illustrate this:
```
struct MyStruct {
    uint256 a;
    uint256 b;
    uint256 c;
}

function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
    MyStruct memory myStruct = MyStruct(_a, _b, _c);
    // Do something with myStruct
}
```
In this example, we have a simple MyStruct data structure with three uint256 fields. The assignValuesToStruct function takes three input parameters _a, _b, and _c, and assigns them to the corresponding fields in a new myStruct variable. This is done using the struct constructor syntax, which creates a new instance of the MyStruct struct with the specified field values.

This approach can be more efficient than assigning values to the struct fields one by one, like this:
```
function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
    MyStruct memory myStruct;
    myStruct.a = _a;
    myStruct.b = _b;
    myStruct.c = _c;
    // Do something with myStruct
}
```
In this example, the values of _a, _b, and _c are assigned to the corresponding fields of the myStruct variable one by one. This can be less efficient than using the struct constructor syntax, because it requires more memory operations to initialize the struct fields.

By using the struct constructor syntax to assign values to the struct fields, we can save gas by reducing the number of memory operations required to create the struct.

```solidity
File: /main/contracts/Lens/PoolLens.sol
295  VTokenBalances({
                vToken: address(vToken),
                balanceOf: balanceOf,
                borrowBalanceCurrent: borrowBalanceCurrent,
                balanceOfUnderlying: balanceOfUnderlying,
                tokenBalance: tokenBalance,
                tokenAllowance: tokenAllowance
            });

329  PoolData memory poolData = PoolData({
            name: venusPool.name,
            creator: venusPool.creator,
            comptroller: venusPool.comptroller,
            blockPosted: venusPool.blockPosted,
            timestampPosted: venusPool.timestampPosted,
            category: venusPoolMetaData.category,
            logoURL: venusPoolMetaData.logoURL,
            description: venusPoolMetaData.description,
            vTokens: vTokenMetadataItems,
            priceOracle: address(comptrollerViewInstance.oracle()),
            closeFactor: comptrollerViewInstance.closeFactorMantissa(),
            liquidationIncentive: comptrollerViewInstance.liquidationIncentiveMantissa(),
            minLiquidatableCollateral: comptrollerViewInstance.minLiquidatableCollateral()
        });
406  VTokenUnderlyingPrice({
                vToken: address(vToken),
                underlyingPrice: priceOracle.getUnderlyingPrice(address(vToken))
            });

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L295

## [G-23] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary


```solidity
File: /main/contracts/Comptroller.sol
128    require(poolRegistry_ != address(0), "invalid pool registry address");

962    require(address(newOracle) != address(0), "invalid price oracle address");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128

```solidity
File: /main/contracts/Pool/PoolRegistry.sol
225     require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

226     require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

257    require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

258    require(input.asset != address(0), "PoolRegistry: Invalid asset address");

259    require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

260    require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

264    require(
            _vTokens[input.comptroller][input.asset] == address(0),
            "PoolRegistry: Market already added for asset comptroller combination"
        );

396    require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");
422    if (address(shortfall_) == address(0)) {

431    if (protocolShareReserve_ == address(0)) {    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L225

```solidity
File: /main/contracts/RiskFund/ProtocolShareReserve.sol
40   require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
41   require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

54   require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

71   require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40

```solidity
File: /main/contracts/RiskFund/ReserveHelpers.sol
40   require(asset != address(0), "ReserveHelpers: Asset address invalid");

52   require(asset != address(0), "ReserveHelpers: Asset address invalid");

53   require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L40

```solidity
File: /main/contracts/RiskFund/RiskFund.sol
80   require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

81   require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

100  require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

111  require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");

127  require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

157  require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80

```solidity
File: /main/contracts/Shortfall/Shortfall.sol
137   require(convertibleBaseAsset_ != address(0), "invalid base asset address");

138   require(address(riskFund_) != address(0), "invalid risk fund address");

180   if (auction.highestBidder != address(0)) {

189   if (auction.highestBidder != address(0)) {

349   require(_poolRegistry != address(0), "invalid address");
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L137

```solidity
File: /main/contracts/VToken.sol
72   require(admin_ != address(0), "invalid admin address");

134  require(spender != address(0), "invalid spender address");

196  require(minter != address(0), "invalid minter address");

626  require(spender != address(0), "invalid spender address");

646  require(spender != address(0), "invalid spender address");

1399  if (shortfall_ == address(0)) {

1408  if (protocolShareReserve_ == address(0)) {    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72

## [G-24]use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: /main/contracts/Comptroller.sol
161   uint256[] memory results = new uint256[](len);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L161



```solidity
File: /main/contracts/Pool/PoolRegistry.sol
306   uint256[] memory newSupplyCaps = new uint256[](1);

307   uint256[] memory newBorrowCaps = new uint256[](1);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L306

## [G‑25] Using bools for storage incurs overhead
```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

### Note: this issue missed from bots.
```solidity
File: /main/contracts/ComptrollerStorage.sol
115    bool internal constant _isComptroller = true;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L115

```solidity
File: /main/contracts/InterestRateModel.sol
10   bool public constant isInterestRateModel = true;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol#L10

```solidity
File: /main/contracts/VTokenInterfaces.sol
142   bool public constant isVToken = true;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L142