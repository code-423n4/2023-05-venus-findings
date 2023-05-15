# GAS OPTIMIZATION

##

## [G-] Refactor the state variables to be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

### Saves 2 Gsset (40000 gas) 

```
     /// @notice Time to wait for next bidder. initially waits for 10 blocks
     uint256 public nextBidderBlockLimit;

    /// @notice Time to wait for first bidder. initially waits for 100 blocks
    uint256 public waitForFirstBidder;

```

- As per docs nextBidderBlockLimit,waitForFirstBidder these state variables only stores the waiting time for bidders.The waiting time not going to too high. 

- So its possible refactor uint256 to uint96. Can saves 2 slots and 40000 gas

- In Solidity, the uint96 type represents an unsigned integer with a range from 0 to 2^96 - 1.The exact possible value to store in uint96 2^96 - 1 is 79,228,162,514,264,337,593,543,950,335.

- 79,228,162,514,264,337,593,543,950,335 seconds ≈ 2,513,031,118.1 years (assuming regular years)

- Uint96 alone more than enough to store 2,513,031,118.1 years

### After refactoring can save 2 slots (2 Gsset)
  
```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Shortfall/Shortfall.sol

    48: address public poolRegistry;
 +  63: uint96 public nextBidderBlockLimit;
    /// @notice Risk fund address
    51: IRiskFund private riskFund;

    /// @notice Minimum USD debt in pool for shortfall to trigger
    54: uint256 public minimumPoolBadDebt;

    /// @notice Incentive to auction participants, initial value set to 1000 or 10%
    57: uint256 private incentiveBps;

    /// @notice Max basis points i.e., 100%
    60: uint256 private constant MAX_BPS = 10000;

    /// @notice Time to wait for next bidder. initially waits for 10 blocks
 -  63: uint256 public nextBidderBlockLimit;

    /// @notice Time to wait for first bidder. initially waits for 100 blocks
 -  66: uint256 public waitForFirstBidder;
 +  66: uint96 public waitForFirstBidder;
    /// @notice base asset contract address
    69: address public convertibleBaseAsset;

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L48-L69

### 1 slot saved (1 gsset) 20000 gas 

Move _isComptroller bool variable bellow the oracle variable. So this will stored within single slot instead of 2 slots 

```solidity
FILE: 2023-05-venus/contracts/ComptrollerStorage.sol

   59: PriceOracle public oracle;
 + 115:   bool internal constant _isComptroller = true;

    /**
     * @notice Multiplier used to calculate the maximum repayAmount when liquidating a borrow
     */
  64: uint256 public closeFactorMantissa;

  101: mapping(address => bool) internal rewardsDistributorExists;

- 115:   bool internal constant _isComptroller = true;


```

##

## [G-] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

- Instances (10)

- Gas Saved : 1200 gas  

calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

At least 120 gas saved for every instances 

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

+ 154: function enterMarkets(address[] calldata vTokens) external override returns (uint256[] memory) {
- 154: function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/VToken.sol

64: string memory name_,
65: string memory symbol_,
69:  RiskManagementInit memory riskManagement,

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L64-L65

```solidity
FILE: 2023-05-venus/contracts/Rewards/RewardsDistributor.sol

166: Exp memory marketBorrowIndex

187: function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external
 onlyComptroller {

198: VToken[] memory vTokens,

199: uint256[] memory supplySpeeds,

200: uint256[] memory borrowSpeeds

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#LL166C9-L166C37

```solidity
FILE: 2023-05-venus/contracts/Pool/PoolRegistry.sol

343: function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL343C5-L343C100

##

## [G-1] Refactor the "for" loop to save gas

### The code should be updated to remove the local variable for vToken and instead directly use VToken(vTokens[i]) within the _addToMarket function call 

As per Remix [Test Reports](https://gist.github.com/sathishpic22/1c92c01937691e12198e4545a72c3c1f) Its possible to save 13 gas for every iterations

The saved gas increased as per iterations count. So can't find the approximate gas savings.   

### BEFORE CHANGE

| GAS | TRANS COST | EXE COST |
|----------|----------|----------|
| 26854 | 23351 | 1639  |

- INSTANCE-1 

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

162: uint256[] memory results = new uint256[](len);
163:        for (uint256 i; i < len; ++i) {
164:            VToken vToken = VToken(vTokens[i]);

165:            _addToMarket(vToken, msg.sender);
166:            results[i] = NO_ERROR;
167:        }

```

### AFTER CHANGE

| GAS | TRANS COST | EXE COST |
|----------|----------|----------|
| 26839 | 23338 | 1626 |

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

   162: uint256[] memory results = new uint256[](len);
   163:        for (uint256 i; i < len; ++i) {
 - 164:            VToken vToken = VToken(vTokens[i]);
 + 165:            _addToMarket(VToken(vTokens[i]), msg.sender);
 - 165:            _addToMarket(vToken, msg.sender);
   166:            results[i] = NO_ERROR;
   167:        }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL162C9-L167C10

- INSTANCE-2 

vTokenSupply only used only once. So need to cache this with stack variable.

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

- 263: uint256 vTokenSupply = VToken(vToken).totalSupply();
  264: Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
+ 265: uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, VToken(vToken).totalSupply(), mintAmount);
- 265: uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L263-L265

- INSTANCE-3

vToken only used once inside the function. Instead of caching can be used directly IERC20Upgradeable function call

```solidity
FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

   223: for (uint256 i; i < marketsCount; ++i) {
 - 224:         VToken vToken = VToken(address(auction.markets[i]));
 - 225:         IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));
 + 225:         IERC20Upgradeable erc20 = IERC20Upgradeable(address(VToken(address(auction.markets[i])).underlying()));
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL223C9-L225C87

- INSTANCE-4

vToken No need to cache 

```solidity
FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

    374: for (uint256 i; i < marketsCount; ++i) {
 -  375:         VToken vToken = auction.markets[i];
 -  376:         auction.marketDebt[vToken] = 0;
 +  376:         auction.marketDebt[auction.markets[i]] = 0;
    377:      }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L374-L377

- INSTANCE-5

```solidity
FILE: 2023-05-venus/contracts/Pool/PoolRegistry.sol

  356: for (uint256 i = 1; i <= _numberOfPools; ++i) {
- 357:          address comptroller = _poolsByID[i];
- 358:          _pools[i - 1] = (_poolByComptroller[comptroller]);
+ 358:          _pools[i - 1] = (_poolByComptroller[_poolsByID[i]]);
  359:      }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL356C9-L359C10

##

## [G-] Caching msg.sender cause more gas

Use msg.sender directly without caching 

```solidity
FILE: 2023-05-venus/contracts/VToken.sol

136: address src = msg.sender;
628: address src = msg.sender;
648: address src = msg.sender;

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L136

##

## [G-] Avoid Emitting State Variables When Stack Variables Are Available

- Instances (2) 

- Gas Saved: 244 gas 

In the instance below, we can emit the calldata value instead of emitting a storage value. This will result in using a cheap CALLDATALOAD instead of an expensive SLOAD

As per Remix [Sample Test](https://gist.github.com/sathishpic22/f294c3b0ccaae13c144ef3858bcd426c)  possible to save 122 gas for every instances 

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Comptroller.sol

  - 709: emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
  + 709: emit NewCloseFactor(oldCloseFactorMantissa, newCloseFactorMantissa);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL709C74-L709C74

```solidity
FILE: 2023-05-venus/contracts/Rewards/RewardsDistributor.sol

  - 268: emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
  + 268: emit ContributorRewardsUpdated(contributor, contributorAccrued);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L268

##

## [G-] Use constants instead of type(uintx).max

type(uint256).max uses more gas in the distribution process and also for each transaction than constant usage

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

262: if (supplyCap != type(uint256).max) {
351: if (borrowCap != type(uint256).max) {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L262

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/VToken.sol

1055:  if (repayAmount == type(uint256).max) {
1314:  startingAllowance = type(uint256).max;
1331:  if (startingAllowance != type(uint256).max) {

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L351

##

## [G-] <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables  

FOR EVERY CALL CAN SAVE 13 GAS


##

## [G-5] Lack of input value checks cause a redeployment if any human/accidental errors

> Instances() 

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables.

If any human/accidental errors happen need to redeploy the contract so this create the huge gas lose 

```solidity


```


```solidity


```

```solidity


```


```solidity


```

```solidity


```


##

## [G-6] Use nested if and, avoid multiple check combinations

> Instances(4)

> Approximate Gas Saved: 36 gas 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas 

```solidity


```


```solidity

```

##

## [G-7] No need to evaluate all expressions to know if one of them is true

> Instances(1) 

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity

```



##

## [G-9] Amounts should be checked for 0 before calling a transfer

> Instances(10) 

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here

```solidity

```

```solidity

```


```solidity 

```


##

## [G-16] Shorthand way to write if / else statement can reduce the deployment cost 

> Instances(7) 

The normal if / else statement can be refactored in a shorthand way to write it:

Increases readability
Shortens the overall SLOC

```solidity

```


```solidity

```


```solidity


```


### Recommended Mitigation 

```solidity

##

## [G-20] Use a more recent version of solidity

CONTEXT:
ALL SCOPE CONTRACTS

- Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
- In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

- in 0.8.19 prevents 

  - Assembler: Avoid duplicating subassembly bytecode where possible.
Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
  - ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
  - SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
  - SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
  - TypeChecker: Also allow external library functions in using for.

##

[G-] Save gas by checking against default WETH address

external call la address check pannama check and value aa immutable ls store panni 2100 gas save pannalam 

You can save a Gcoldsload (2100 gas) in the address provider, plus the 100 gas overhead of the external call, for every receive(), by creating an immutable DEFAULT_WETH variable which will store the initial WETH address, and change the require statement to be: require(msg.ender == DEFAULT_WETH || msg.sender == <etc>).






[G‑01]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	3	-
[G‑02]	Structs can be packed into fewer storage slots	3	-
[G‑03]	Using storage instead of memory for structs/arrays saves gas	6	25200
[G‑04]	State variables should be cached in stack variables rather than re-reading them from storage	22	2134
[G‑05]	Multiple accesses of a mapping/array should use a local variable cache	13	546
[G‑06]	The result of function calls should be cached rather than re-calling the function	1	-
[G‑07]	internal functions only called once can be inlined to save gas	19	380
[G‑08]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	3	255
[G‑09]	<array>.length should not be looked up in every loop of a for-loop	3	9
[G‑10]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	37	2220
[G‑11]	require()/revert() strings longer than 32 bytes cost extra gas	61	-
[G‑12]	Optimize names to save gas	16	352
[G‑13]	Using bools for storage incurs overhead	3	51300
[G‑14]	>= costs less gas than >	1	3
[G‑15]	internal functions not called by the contract should be removed to save deployment gas	2	-
[G‑16]	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	1	5
[G‑17]	Splitting require() statements that use && saves gas	4	12
[G‑18]	Using private rather than public for constants, saves gas	8	-
[G‑19]	Division by two should use bit shifting	1	20
[G‑20]	Stack variable used as a cheaper cache for a state variable is only used once	10	30
[G‑21]	require() or revert() statements that check input arguments should be at the top of the function	12	-
[G‑22]	Use custom errors rather than revert()/require() strings to save gas	99	-
[G‑23]	Functions guaranteed to revert when called by normal users can be marked payable	22	462
[G‑24]	Constructors can be marked payable	11	231