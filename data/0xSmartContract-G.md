### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Remove or replace unused state variables | 1 |
| [G-02] |"Zero Value" deficient controls |2 |
| [G-03] |Duplicated require()/if() checks should be refactored to a modifier or function | 5 |
| [G-04] |Using msg.sender directly instead of caching is gas optimizedt |4|
| [G-05] |if () / require() statements that check input arguments should be at the top of the function | 2 |
| [G-06] |Use ``assembly`` to write _address storage values_  | 25 |
| [G-07] |Use nested if and, avoid multiple check combinations  | 12 |
| [G-08] |Using ``delete`` instead of setting ``address(0)`` saves gas | 1 |
| [G-09] |Do not calculate constants variables | 1 |
| [G-10] |Sort Solidity operations using short-circuit mode | 1 |
| [G-11] |Use assembly to check for address(0) | 41 |
| [G-12] |Use ``require`` instead of ``assert`` | 1 |
| [G-13] |Empty blocks should be removed or emit somethings |1 |

Total 13 issues


### [G-01] Remove or replace unused state variables

Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it's assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

1 result - 1 file:

```solidity
contracts\ExponentialNoError.sol:

  22:     uint256 internal constant halfExpScale = expScale / 2;

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22

### [G-02] "Zero Value" deficient controls

The variables ``baseRatePerYear`` and ``multiplierPerYear`` are ``immutable`` state variables that are updated only once in the constructor. 

As seen below, ``baseRatePerYear`` and ``multiplierPerYear`` values entered as arguments in the constructor are two important arguments for creating **interest rate model** for the project. The absence of zero control of these values means a new deploy cost.

From the perspective of the project, it means that the **current debt ratio per block** will be calculated as zero and the project will incur a loss.

My suggestion is to add a 'zero' check for these values.

2 results - 1 file:
```solidity
contracts\WhitePaperInterestRateModel.sol:

  36:     constructor(uint256 baseRatePerYear, uint256 multiplierPerYear) {
  37:         baseRatePerBlock = baseRatePerYear / blocksPerYear;
  38:         multiplierPerBlock = multiplierPerYear / blocksPerYear;
  39: 
  40:         emit NewInterestParams(baseRatePerBlock, multiplierPerBlock);
  41:     }

````

### [G-03] Duplicated require()/if() checks should be refactored to a modifier or function

5 results - 2 files:

```solidity
contracts\RiskFund\ReserveHelpers.sol:

  39          require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
  40:         require(asset != address(0), "ReserveHelpers: Asset address invalid");


  51          require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
  52:         require(asset != address(0), "ReserveHelpers: Asset address invalid");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L39-L40
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L51-L52



```solidity
contracts\VToken.sol:

  133      function approve(address spender, uint256 amount) external override returns (bool) {
  134:         require(spender != address(0), "invalid spender address");


  625      function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
  626:         require(spender != address(0), "invalid spender address");
 

  645      function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
  646:         require(spender != address(0), "invalid spender address");


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L134
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L626
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L646


**Recommendation:**

You can consider adding a modifier like below.

```solidity
 modifer checkZeroAddress () {
        require(spender != address(0), "invalid spender address");
        _;
 }

```

### [G-04] Using msg.sender directly instead of caching is gas optimized

4 results - 2 files:

```solidity
contracts\VToken.sol:
  133:     function approve(address spender, uint256 amount) external override returns (bool) {
  134:         require(spender != address(0), "invalid spender address");
  135: 
- 136:         address src = msg.sender;
- 137:         transferAllowances[src][spender] = amount;
+ 137:         transferAllowances[msg.sender][spender] = amount;
- 138:         emit Approval(src, spender, amount);
+ 138:         emit Approval(msg.sender, spender, amount);
  139:         return true;
  140:     }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L136-L138

```diff
contracts\VToken.sol:

  625:     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
  626:         require(spender != address(0), "invalid spender address");
  627: 
- 628:         address src = msg.sender;
  629:         uint256 newAllowance = transferAllowances[src][spender];
  630:         newAllowance += addedValue;
- 631:         transferAllowances[src][spender] = newAllowance;
+ 631:         transferAllowances[msg.sender][spender] = newAllowance;
  632: 
- 633:         emit Approval(src, spender, newAllowance);
+ 633:         emit Approval(msg.sender, spender, newAllowance);
  634:         return true;
  635:     }
  636: 

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L631-L633


```diff
contracts\VToken.sol:

  645:     function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
  646:         require(spender != address(0), "invalid spender address");
  647: 
- 648:         address src = msg.sender;
  649:         uint256 currentAllowance = transferAllowances[src][spender];
  650:         require(currentAllowance >= subtractedValue, "decreased allowance below zero");
  651:         unchecked {
  652:             currentAllowance -= subtractedValue;
  653:         }
  654: 
- 655:         transferAllowances[src][spender] = currentAllowance;
+ 655:         transferAllowances[msg.sender][spender] = currentAllowance;
  656: 
- 657:         emit Approval(src, spender, currentAllowance);
+ 657:         emit Approval(msg.sender, spender, currentAllowance);
  658:         return true;
  659:     }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L655-L657


```diff
contracts\Comptroller.sol:

  578:     function healAccount(address user) external {
  579:         VToken[] memory userAssets = accountAssets[user];
  580:         uint256 userAssetsCount = userAssets.length;
  581: 
- 582:         address liquidator = msg.sender;
  583:         // We need all user's markets to be fresh for the computations to be correct
  584:         for (uint256 i; i < userAssetsCount; ++i) {
  585:             userAssets[i].accrueInterest();
  586:             oracle.updatePrice(address(userAssets[i]));
  587:         }
  588: 
  589:         AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(user, _getLiquidationThreshold);
  590: 
  591:         if (snapshot.totalCollateral > minLiquidatableCollateral) {
  592:             revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
  593:         }
  594: 
  595:         if (snapshot.shortfall == 0) {
  596:             revert InsufficientShortfall();
  597:         }
  598: 
  599:         // percentage = collateral / (borrows * liquidation incentive)
  600:         Exp memory collateral = Exp({ mantissa: snapshot.totalCollateral });
  601:         Exp memory scaledBorrows = mul_(
  602:             Exp({ mantissa: snapshot.borrows }),
  603:             Exp({ mantissa: liquidationIncentiveMantissa })
  604:         );
  605: 
  606:         Exp memory percentage = div_(collateral, scaledBorrows);
  607:         if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
  608:             revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
  609:         }
  610: 
  611:         for (uint256 i; i < userAssetsCount; ++i) {
  612:             VToken market = userAssets[i];
  613: 
  614:             (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);
  615:             uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);
  616: 
  617:             // Seize the entire collateral
  618:             if (tokens != 0) {
- 619:                 market.seize(liquidator, user, tokens);
+ 619:                 market.seize(msg.sender, user, tokens);
  620:             }
  621:             // Repay a certain percentage of the borrow, forgive the rest
  622:             if (borrowBalance != 0) {
- 623:                 market.healBorrow(liquidator, user, repaymentAmount);
+ 623:                 market.healBorrow(msg.sender, user, repaymentAmount);
  624:             }
  625:         }
  626:     }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L618-L623


### [G-05] if () / require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

2 results - 1 files:
```solidity
contracts\VToken.sol:

  1177:     function _addReservesFresh(uint256 addAmount) internal returns (uint256) {
  1178:         // totalReserves + actualAddAmount
  1179:         uint256 totalReservesNew;
  1180:         uint256 actualAddAmount;
  1181: 
  1182:         // We fail gracefully unless market's block number equals current block number
  1183:         if (accrualBlockNumber != _getBlockNumber()) {
  1184:             revert AddReservesFactorFreshCheck(actualAddAmount);
  1185:         }

```

```diff
contracts\VToken.sol:

  1177:     function _addReservesFresh(uint256 addAmount) internal returns (uint256) {
+ 1182:         // We fail gracefully unless market's block number equals current block number
+ 1183:         if (accrualBlockNumber != _getBlockNumber()) {
+ 1184:             revert AddReservesFactorFreshCheck(actualAddAmount);
+ 1185:         }
  1178:         // totalReserves + actualAddAmount
  1179:         uint256 totalReservesNew;
  1180:         uint256 actualAddAmount;
  1181: 
- 1182:         // We fail gracefully unless market's block number equals current block number
- 1183:         if (accrualBlockNumber != _getBlockNumber()) {
- 1184:             revert AddReservesFactorFreshCheck(actualAddAmount);
- 1185:         }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1177-L1193


```solidity
contracts\VToken.sol:
  1200:     function _reduceReservesFresh(uint256 reduceAmount) internal {
  1201:         // totalReserves - reduceAmount
  1202:         uint256 totalReservesNew;
  1203: 
  1204:         // We fail gracefully unless market's block number equals current block number
  1205:         if (accrualBlockNumber != _getBlockNumber()) {
  1206:             revert ReduceReservesFreshCheck();
  1207:         }
  1208: 
  1209:         // Fail gracefully if protocol has insufficient underlying cash
  1210:         if (_getCashPrior() < reduceAmount) {
  1211:             revert ReduceReservesCashNotAvailable();
  1212:         }
  1213: 
  1214:         // Check reduceAmount ≤ reserves[n] (totalReserves)
  1215:         if (reduceAmount > totalReserves) {
  1216:             revert ReduceReservesCashValidation();
  1217:         }
  1218: 
  1219:         /////////////////////////
  1220:         // EFFECTS & INTERACTIONS
  1221:         // (No safe failures beyond this point)
  1222: 
  1223:         totalReservesNew = totalReserves - reduceAmount;
  1224: 
  1225:         // Store reserves[n+1] = reserves[n] - reduceAmount
  1226:         totalReserves = totalReservesNew;
  1227: 

```


```diff
contracts\VToken.sol:
  1200:     function _reduceReservesFresh(uint256 reduceAmount) internal {
+ 1214:         // Check reduceAmount ≤ reserves[n] (totalReserves)
+ 1215:         if (reduceAmount > totalReserves) {
+ 1216:             revert ReduceReservesCashValidation();
+ 1217:         }
- 1201:         // totalReserves - reduceAmount
- 1202:         uint256 totalReservesNew;
  1203: 
  1204:         // We fail gracefully unless market's block number equals current block number
  1205:         if (accrualBlockNumber != _getBlockNumber()) {
  1206:             revert ReduceReservesFreshCheck();
  1207:         }
  1208: 
  1209:         // Fail gracefully if protocol has insufficient underlying cash
  1210:         if (_getCashPrior() < reduceAmount) {
  1211:             revert ReduceReservesCashNotAvailable();
  1212:         }
  1213: 
- 1214:         // Check reduceAmount ≤ reserves[n] (totalReserves)
- 1215:         if (reduceAmount > totalReserves) {
- 1216:             revert ReduceReservesCashValidation();
- 1217:         }
  1218: 
  1219:         /////////////////////////
  1220:         // EFFECTS & INTERACTIONS
  1221:         // (No safe failures beyond this point)
+ 1201:         // totalReserves - reduceAmount
+ 1202:         uint256 totalReservesNew;
  1222: 
  1223:         totalReservesNew = totalReserves - reduceAmount;
  1224: 
  1225:         // Store reserves[n+1] = reserves[n] - reduceAmount
  1226:         totalReserves = totalReservesNew;
  1227: 

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1200-L1236


### [G-06] Use ``assembly`` to write _address storage values_ 

There are 25 examples related to the subject in the contracts within the scope, and with this optimization, ~1.5 thousand gas savings and a serious deployment cost savings will be achieved.

```solidity
contracts\BaseJumpRateModelV2.sol:

  74:         accessControlManager = accessControlManager_;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L74


```solidity
contracts\Comptroller.sol:

  965:         oracle = newOracle;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L965


```solidity
contracts\VToken.sol:

  1144:         comptroller = newComptroller;

  1259:         interestRateModel = newInterestRateModel;

  1390:         underlying = underlying_;

  1402          address oldShortfall = shortfall;

  1403:         shortfall = shortfall_;

  1412:         protocolShareReserve = protocolShareReserve_;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1144


```solidity
contracts\Pool\PoolRegistry.sol:
 
  175:         vTokenFactory = vTokenFactory_;

  176:         jumpRateFactory = jumpRateFactory_;

  177:         whitePaperFactory = whitePaperFactory_;

  426:         shortfall = shortfall_;

  435:         protocolShareReserve = protocolShareReserve_;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L175-L177


```solidity
contracts\Rewards\RewardsDistributor.sol:

  117:         comptroller = comptroller_;

  118:         rewardToken = rewardToken_;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L117-L118


```solidity
contracts\RiskFund\ProtocolShareReserve.sol:
 
  45:         protocolIncome = _protocolIncome;

  46:         riskFund = _riskFund;

  56:         poolRegistry = _poolRegistry;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L45-L46


```solidity
contracts\RiskFund\RiskFund.sol:
  
   88:         pancakeSwapRouter = pancakeSwapRouter_;
 
   90:         convertibleBaseAsset = convertibleBaseAsset_;

  102:         poolRegistry = _poolRegistry;

  118:         shortfall = shortfallContractAddress_;

  129:         pancakeSwapRouter = pancakeSwapRouter_;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L88


```solidity
contracts\Shortfall\Shortfall.sol:

  145:         convertibleBaseAsset = convertibleBaseAsset_;

  146:         riskFund = riskFund_;

  351:         poolRegistry = _poolRegistry;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L145-L146


**Recommendation Code:**
```diff
contracts\Shortfall\Shortfall.sol:

- 145:         convertibleBaseAsset = convertibleBaseAsset_;
+                  assembly {                      
+                      sstore(convertibleBaseAsset.slot, convertibleBaseAsset_)
+                  }       

```

### [G-07] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

12 results - 4 files:

```solidity
contracts\Comptroller.sol:

  755:         if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755


```solidity
contracts\VToken.sol:

  837:         if (redeemTokens == 0 && redeemAmount > 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837


```solidity
contracts\Lens\PoolLens.sol:

  462:         if (deltaBlocks > 0 && borrowSpeed > 0) {
  483:         if (deltaBlocks > 0 && supplySpeed > 0) {
  506:         if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
  526:         if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462


```solidity
contracts\Rewards\RewardsDistributor.sol:

  261:         if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
  348:         if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {
  386:         if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
  418:         if (amount > 0 && amount <= rewardTokenRemaining) {
  435:         if (deltaBlocks > 0 && supplySpeed > 0) {
  463:         if (deltaBlocks > 0 && borrowSpeed > 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261


### [G-08] Using ``delete`` instead of setting ``address(0)`` saves gas

1 result - 1 file:
```diff
contracts\Shortfall\Shortfall.sol:

- 423:         auction.highestBidder = address(0);
+ 423:         delete auction.highestBidder;
  
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L423


**Proof Of Concepts:**
contract Example {
    address public myAddress = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

    //  use for empty value Slot:     23450 gas
    //  use for non empty value Slot: 21450 gas
    function reset() public {
        delete myAddress;
    }


    // use for empty value Slot:     23497 gas
    // use for non empty value Slot: 23497 gas
    function setToZero() public {
        myAddress = address(0);
    }
}


### [G-09] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

1 result - 1 file:

| Deploy gas saved    |  before   |  after  | gas saved|
|:-------------------------|:---------:|:-------:|:--------:|
|21k |   532  |  351 |    181   |

```solidity
contracts\ExponentialNoError.sol:

 22:     uint256 internal constant halfExpScale = expScale / 2;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22


```diff
contracts\ExponentialNoError.sol:

- 22:     uint256 internal constant halfExpScale = expScale / 2;

+         // halfExpScale = expScale / 2
+ 22:     uint256 internal constant halfExpScale = 5e17;

```


### [G-10] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses ```OR/AND``` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

1 result - 1 file:

```solidity
contracts\Shortfall\Shortfall.sol:

  164:         require(
  165:             (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
  166:                 ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
  167:                     (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
  168:                 (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
  169:                     ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
  170:                         (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
  171:             "your bid is not the highest"
  172:         );

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L164-L172


### [G-11] Use assembly to check for address(0)

41 results - 9 files: (41 * 6 gas = 246 gas)
```solidity
contracts\VToken.sol:

  1399:         if (shortfall_ == address(0)) {
  1400:             revert ZeroAddressNotAllowed();
  1401:         }

  1407      function _setProtocolShareReserve(address payable protocolShareReserve_) internal {
  1408:         if (protocolShareReserve_ == address(0)) {
  1409:             revert ZeroAddressNotAllowed();
  1410:         }
  1411          address oldProtocolShareReserve = address(protocolShareReserve);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1399


```solidity
contracts\Pool\PoolRegistry.sol:

  422:         if (address(shortfall_) == address(0)) {
  423:             revert ZeroAddressNotAllowed();
  424:         }

  430      function _setProtocolShareReserve(address payable protocolShareReserve_) internal {
  431:         if (protocolShareReserve_ == address(0)) {
  432:             revert ZeroAddressNotAllowed();
  433:         }
  434          address oldProtocolShareReserve = protocolShareReserve;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422


```solidity
contracts\BaseJumpRateModelV2.sol:

  72:         require(address(accessControlManager_) != address(0), "invalid ACM address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L72

```solidity
contracts\Comptroller.sol:

  128:         require(poolRegistry_ != address(0), "invalid pool registry address");

  962:         require(address(newOracle) != address(0), "invalid price oracle address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128

```solidity
contracts\VToken.sol:

   72:         require(admin_ != address(0), "invalid admin address");

  134:         require(spender != address(0), "invalid spender address");

  196:         require(minter != address(0), "invalid minter address");

  626:         require(spender != address(0), "invalid spender address");

  646:         require(spender != address(0), "invalid spender address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72

```solidity
contracts\Pool\PoolRegistry.sol:

  225:         require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

  226:         require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");

  257:         require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

  258:         require(input.asset != address(0), "PoolRegistry: Invalid asset address");

  259:         require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

  260:         require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L225-L226


```solidity
contracts\Proxy\UpgradeableBeacon.sol:

  8:         require(implementation_ != address(0), "Invalid implementation address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L8


```solidity
contracts\RiskFund\ProtocolShareReserve.sol:

  40:         require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

  41:         require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

  54:         require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

  71:         require(asset != address(0), "ProtocolShareReserve: Asset address invalid");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40-L41


```solidity
contracts\RiskFund\ReserveHelpers.sol:

  40:         require(asset != address(0), "ReserveHelpers: Asset address invalid");

  52:         require(asset != address(0), "ReserveHelpers: Asset address invalid");

  53:         require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");

  54          require(
  55:             PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
  56              "ReserveHelpers: The pool doesn't support the asset"

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L40


```solidity
contracts\RiskFund\RiskFund.sol:

   80:         require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

   81:         require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

  100:         require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

  111:         require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");

  127:         require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

  157:         require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80-L81

```solidity
contracts\Shortfall\Shortfall.sol:

  139:         require(convertibleBaseAsset_ != address(0), "invalid base asset address");

  140:         require(address(riskFund_) != address(0), "invalid risk fund address");

  215          require(
  216:             block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),
  217              "waiting for next bidder. cannot close auction"

  351:         require(_poolRegistry != address(0), "invalid address");

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L137-L139


```diff
contracts\Shortfall\Shortfall.sol:

  350:     function updatePoolRegistry(address _poolRegistry) external onlyOwner {
- 351:         require(_poolRegistry != address(0), "invalid address");
+              assembly {
+                 if iszero(_poolRegistry) {
+                    mstore(0x00, "invalid address")
+                    revert(0x00, 0x20)
+                 }
+              }
+          } 

```
### [G-12] Use ``require`` instead of ``assert``

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

1 result - 1 file:
```solidity
contracts\Comptroller.sol:

  224          // We *must* have found the asset in the list or our redundant data structure is broken
  225:         assert(assetIndex < len);
  226  

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L225


### [G-13] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

1 result - 1 file:
```solidity
contracts\JumpRateModelV2.sol:
  12:     constructor(
  13:         uint256 baseRatePerYear,
  14:         uint256 multiplierPerYear,
  15:         uint256 jumpMultiplierPerYear,
  16:         uint256 kink_,
  17:         IAccessControlManagerV8 accessControlManager_
  18:     )
  19:         BaseJumpRateModelV2(baseRatePerYear, multiplierPerYear, jumpMultiplierPerYear, kink_, accessControlManager_)
  20:     /* solhint-disable-next-line no-empty-blocks */
  21:     {
  22: 
  23:     }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L12-L23