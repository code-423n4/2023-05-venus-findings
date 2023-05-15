## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate | 3 | - |
| [G-02] | Structs can be packed into fewer storage slots | 1 | - |
| [G-03] | Using storage instead of memory for structs/arrays saves gas | 6 | - |
| [G-04] | Multiple accesses of a mapping/array should use a local variable cache | 9 | - |
| [G-05] | internal functions only called once can be inlined to save gas | 9 | - |
| [G-06] | Using private rather than public for constants, saves gas | 1 | - |
| [G-07] | Use a more recent version of solidity | 28 | - |
| [G-08] | <X> += <Y> Cost more gas than <X> = <X> + <Y> for state variables or ( -= ) | 2 | - |
| [G-09] | Using fixed bytes is cheaper than using string | 9 | - |
| [G-10] | Not using the named return variable when a function returns, wastes deployment gas | 2 | - |
| [G-11] | Can make the variable outside the loop to save gas | 6 | - |
| [G-12] |  Use assembly to check for address(0) | 5 | - |
| [G-13] | Store using Struct over multiple mappings | 3 | - |
| [G-14] | Empty blocks should be removed or emit something | 1 | - |
| [G-15] | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS | 5 | - |
| [G-16] | Remove the initializer modifier | 1 | - |
| [G-17] | Use constants instead of type(uintx).max | 5 | - |
| [G-18] |  Use nested if and, avoid multiple check combinations | 4 | - |
| [G-19] | Do not calculate  constant. | 1 | - |
| [G-20] | Minimize external calls | more file | - | 



## Gas Optimizations  

## [G-1] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Details

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.


```solidity
file: /contracts/Pool/PoolRegistry.sol

98      mapping(address => VenusPool) private _poolByComptroller;


103      mapping(address => mapping(address => address)) private _vTokens;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L98-L103


```solidity
File:  /contracts/VTokenInterfaces.sol

107       mapping(address => uint256) internal accountTokens;

           // Approved token transfer amounts on behalf of others
110      mapping(address => mapping(address => uint256)) internal transferAllowances;

         // Mapping of account addresses to outstanding borrow balances
113      mapping(address => BorrowSnapshot) internal accountBorrows;



```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L107-L113


```solidity
file: /contracts/RiskFund/ReserveHelpers.sol

13      mapping(address => uint256) internal assetsReserves;
   
        // Store the asset's reserve per pool in the ProtocolShareReserve.
        // Comptroller(pool) -> Asset -> amount
17       mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L13-L17



## [G-2] Structs can be packed into fewer storage slots


Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings



```solidity
file: /contracts/Factories/VTokenProxyFactory.sol

11     struct VTokenArgs {
        address underlying_;
        ComptrollerInterface comptroller_;
        InterestRateModel interestRateModel_;
        uint256 initialExchangeRateMantissa_;
        string name_;
        string symbol_;
        uint8 decimals_;
        address admin_;
        AccessControlManager accessControlManager_;
        VTokenInterface.RiskManagementInit riskManagement;
        address beaconAddress;
        uint256 reserveFactor;
24    }

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L11-L24



## [G-3]   Using storage instead of memory for structs/arrays saves gas


When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
file: /contracts/Comptroller.sol

687      VToken[] memory borrowMarkets = accountAssets[borrower];

1059     VToken[] memory assetsIn = accountAssets[account];

1304     VToken[] memory assets = accountAssets[account];

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L687


```solidity 
file: /contracts/VToken.sol

1441       BorrowSnapshot memory borrowSnapshot = accountBorrows[account];

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1441


```solidity
file: /contracts/Shortfall/Shortfall.sol

219         uint256[] memory marketsDebt = new uint256[](marketsCount);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L219


```solidity
file: /contracts/Pool/PoolRegistry.sol

345       VenusPoolMetaData memory oldMetadata = metadata[comptroller];

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L345


## [G-4]   Multiple accesses of a mapping/array should use a local variable cache


The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata



```solidity
file: /contracts/VToken.sol
/// @audit  accountBorrows[borrower] on line 424

425        accountBorrows[borrower].interestIndex = borrowIndex;

/// @audit  accountBorrows[borrower] on line 908

909        accountBorrows[borrower].interestIndex = borrowIndex;


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L424

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L908


```solidity
file: /contracts/Lens/PoolLens.sol
/// @audit aaddress(markets[i]) on line 265


267        VToken(address(markets[i])).badDebt() *
          priceOracle.getUnderlyingPrice(address(markets[i]));

/// @audit   address(markets[i]) on line 421, 423 and 427 

428           updateMarketSupplyIndex(address(markets[i]), rewardsDistributor, supplyState);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L265-L267

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L421


```solidity
file: /contracts/Shortfall/Shortfall.sol

/// @audit  auction.marketDebt[auction.markets[i]]  on line 181, 186 and 190

193           erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);

/// @audit auction.marketDebt[auction.markets[i]]  on line 228

232           erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L193


```solidity
file: 

/// @audit poolReserves[comptroller] on line 192 

193         poolReserves[comptroller] = poolReserves[comptroller] - amount;

/// @audit   poolsAssetsReserves[comptroller][underlyingAsset] on line 236 

250         poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L193


```solidity
file: /contracts/RiskFund/ProtocolShareReserve.sol

/// @audit poolsAssetsReserves[comptroller][asset] on line 72 

75        poolsAssetsReserves[comptroller][asset] -= amount;    

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L75



## [G-5] internal functions only called once can be inlined to save gas


Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.


```solidity
file: /contracts/ExponentialNoError.sol

38        function mul_ScalarTruncate(Exp memory a, uint256 scalar) internal pure returns (uint256) {


47     function mul_ScalarTruncateAddUInt(
         Exp memory a,
         uint256 scalar,
         uint256 addend
51     ) internal pure returns (uint256) {


59       function lessThanExp(Exp memory left, Exp memory right) internal pure returns (bool) {

63       function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {

68       function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {

153      function fraction(uint256 a, uint256 b) internal pure returns (Double memory) {           


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L38


```solidity 
file: /contracts/BaseJumpRateModelV2.sol

172      function _getBorrowRate(
         uint256 cash,
         uint256 borrows,
         uint256 reserves
176     ) internal view returns (uint256) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L172-L176


```solidity
file: /contracts/MaxLoopsLimitHelper.sol

25         function _setMaxLoopsLimit(uint256 limit) internal {

39       function _ensureMaxLoops(uint256 len) internal view {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25


## [G-6] Using private rather than public for constants, saves gas


### Detials

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table


```solidity
file: /contracts/Comptroller.sol

27       address public immutable poolRegistry;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L27


## [G-7] USE A MORE RECENT VERSION OF SOLIDITY

### NOTE: this version of solidity is smaller than current version, its exist in every file. 

```solidity
file: /contracts/Comptroller.sol

2    pragma solidity 0.8.13;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L2


## [G-8]  <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

 AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
file: /contracts/RiskFund/ReserveHelpers.sol

/// @audit the assetsReserves  is state variable

66       assetsReserves[asset] += balanceDifference;

///  @audit the  poolsAssetsReserves  is state variable               
67                poolsAssetsReserves[comptroller][asset] += balanceDifference;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L66

## [G-9]   Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.


```solidity
file: /contracts/Lens/PoolLens.sol

22       string category;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L22


```solidity
file: /contracts/BaseJumpRateModelV2.sol

55         error Unauthorized(address sender, address calledContract, string methodSignature);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L55



## [G-10]   Not using the named return variable when a function returns, wastes deployment gas


```solidity
file: /contracts/ExponentialNoError.sol

/// @audit uint224(n) do not use uint 224 with (n)
63     function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {
64        require(n < 2**224, errorMessage);
65        return uint224(n);

/// @audit uint32(n) do not use uint32 with (n)

68      function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
69         require(n < 2**32, errorMessage);
70         return uint32(n);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L63


## [G-11]  Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas


```solidity
file: /contracts/Comptroller.sol
/// @audit the  rewardToken should make
1123       address rewardToken = address(rewardsDistributors[i].rewardToken());

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1123


```solidity
file: /contracts/Lens/PoolLens.sol

431       uint256 borrowReward = calculateBorrowerReward(

438       uint256 supplyReward = calculateSupplierReward(

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L431


```solidity
file: /contracts/Shortfall/Shortfall.sol

390            uint256 marketBadDebt = vTokens[i].badDebt();

493            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L390


```solidity
file: /contracts/Pool/PoolRegistry.sol

357                address comptroller = _poolsByID[i];

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L357


## [G-12] Use assembly to check for address(0)


```solidity
file: /contracts/VToken.sol
1408        if (protocolShareReserve_ == address(0)) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1408


```solidity
file: /contracts/Shortfall/Shortfall.sol

180       if (auction.highestBidder != address(0)) {

189       if (auction.highestBidder != address(0)) {    

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L180

```solidity
file: /contracts/Pool/PoolRegistry.sol

422          if (address(shortfall_) == address(0)) {

431         if (protocolShareReserve_ == address(0)) {    

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422


## [G-13]  Store using Struct over multiple mappings


```solidity
file: /contracts/Rewards/RewardsDistributor.sol

32     mapping(address => uint256) public rewardTokenAccrued;
35    mapping(address => uint256) public rewardTokenBorrowSpeeds;


41     mapping(address => RewardToken) public rewardTokenBorrowState;
44    mapping(address => uint256) public rewardTokenContributorSpeeds;


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L32



```solidity
file: /contracts/ComptrollerStorage.sol

86       mapping(address => uint256) public borrowCaps;
92       mapping(address => uint256) public supplyCaps;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L86


## [G-14]  Empty blocks should be removed or emit something


```solidity
file: /contracts/JumpRateModelV2.sol

21        {
22
23        }


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L21-L23


## [G-15]  USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

```solidity
file: /contracts/Comptroller.sol

161           uint256[] memory results = new uint256[](len);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L161



```solidity
file: /contracts/Shortfall/Shortfall.sol

219          uint256[] memory marketsDebt = new uint256[](marketsCount);

386          uint256[] memory marketsDebt = new uint256[](marketsCount);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L219


```solidity
file: /contracts/Pool/PoolRegistry.sol

306    uint256[] memory newSupplyCaps = new uint256[](1);
307        uint256[] memory newBorrowCaps = new uint256[](1);


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L306


## [G-16] Remove the initializer modifier

If we can just ensure that the initialize() function could only be called from within the constructor, we shouldn’t need to worry about it getting called again.

```solidity
file: /contracts/Comptroller.sol

138      function initialize(uint256 loopLimit, address accessControlManager) external initializer {


```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L138


## [G-17] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/Comptroller.sol

262           if (supplyCap != type(uint256).max) {

351          if (borrowCap != type(uint256).max) {    

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L262



```solidity
file: /contracts/VToken.sol

1055          if (repayAmount == type(uint256).max) {

1314          startingAllowance = type(uint256).max;    

1331         if (startingAllowance != type(uint256).max) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1055


## [G-18]  Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
file: /contracts/Comptroller.sol

755           if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755



```solidity
file: /contracts/VToken.sol

837          if (redeemTokens == 0 && redeemAmount > 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837



```solidity
file: /contracts/Lens/PoolLens.sol

462          if (deltaBlocks > 0 && borrowSpeed > 0) {

506          if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462


## [G-19] Do not calculate  constant.

```solidity
file: /contracts/ExponentialNoError.sol

22      uint256 internal constant halfExpScale = expScale / 2;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22


## [G-20] Minimize external calls

 External calls to other contracts are expensive in terms of gas. Try to minimize external calls by using in-line code or caching data in memory.

 ### External call is find in more file 
