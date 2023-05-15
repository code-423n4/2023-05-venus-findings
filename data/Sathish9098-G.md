# GAS OPTIMIZATION

| Issue Count | Issues | Instances | Gas Saved|
|-----------------|-----------------|-----------------|-----------------|
| [G-1] |  Refactor the state variables to be packed into fewer storage slots |  3 |  60000 |
| [G-2] | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS  |  10 |  3000 |
| [G-3] | Refactor the "for" loop to save gas  |  5 |  Depends number of iterations |
| [G-4] | IF’s/require() statements that check input arguments should be at the top of the function  |  1 |  2100 |
| [G-5] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead  |  4 |  120 |
| [G-6] | Caching msg.sender cause more gas  |  3 |  - |
| [G-7] | Avoid Emitting State Variables When Stack Variables Are Available|  2 |  244 |
| [G-8] | Use constants instead of type(uintx).max  |  5 |  - |
| [G-9] | Lack of input value checks cause a redeployment if any human/accidental errors  |  9 |  - |
| [G-10] | Use nested if and, avoid multiple check combinations  |  10 |  90 |
| [G-11] | No need to evaluate all expressions to know if one of them is true  |  4 |  - |
| [G-12] | Amounts should be checked for 0 before calling a transfer  |  7 |  - |
| [G-13] | Use a more recent version of solidity  |  - |  - |
| [G-14] | Non-usage of specific imports  |  - |  - |
| [G-15] | Shorten the array rather than copying to a new one |  13 |  - |

##

## [G-1] Refactor the state variables to be packed into fewer storage slots

- Instances (3)

- Gas Saved : 60000 gas (3 Gsset )

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.

### 1 slot saved (1 gsset) 20000 gas 

Move _isComptroller bool variable bellow the oracle variable. So this will stored within single slot instead of 2 slots 

```solidity
FILE: 2023-05-venus/contracts/ComptrollerStorage.sol

The lines shortened for better understanding 

   59: PriceOracle public oracle;
 + 115: bool internal constant _isComptroller = true;

    /**
     * @notice Multiplier used to calculate the maximum repayAmount when liquidating a borrow
     */
  64: uint256 public closeFactorMantissa;

  101: mapping(address => bool) internal rewardsDistributorExists;

- 115:   bool internal constant _isComptroller = true;

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L59-L115

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

##

## [G-2] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

- Instances (10)

- Gas Saved : 3000 gas  

calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

At least 300 gas saved for every instances 

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

## [G-3] Refactor the "for" loop to save gas

- Instances (5)

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

## [G-4] IF’s/require() statements that check input arguments should be at the top of the function

- Instances (1)

- Gas Saved : 2100 gas 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Shortfall/Shortfall.sol

MAX_BPS constant check should come first 

+ 163: require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
  161: require(_isStarted(auction), "no on-going auction");
  162: require(!_isStale(auction), "auction is stale, restart it");
- 163: require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL161C9-L163C78

##

## [G-5] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

- Instances(4)

- Gas Saved : 120 gas 

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

29:  uint224 public constant rewardTokenInitialIndex = 1e36;
128: uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
433: uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
461: uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L29

##

## [G-6] Caching msg.sender cause more gas

- Instances (3)

Use msg.sender directly without caching 

```solidity
FILE: 2023-05-venus/contracts/VToken.sol

136: address src = msg.sender;
628: address src = msg.sender;
648: address src = msg.sender;

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L136

##

## [G-7] Avoid Emitting State Variables When Stack Variables Are Available

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

## [G-8] Use constants instead of type(uintx).max

- Instances(5)

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

## [G-9] Lack of input value checks cause a redeployment if any human/accidental errors

- Instances(5) 

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables.

If any human/accidental errors happen need to redeploy the contract so this create the huge gas lose 

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Comptroller.sol

loopLimit is not checked before set _setMaxLoopsLimit. loopLimit can be zero value. Should be checked before set _setMaxLoopsLimit

function initialize(uint256 loopLimit, address accessControlManager) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager);

        _setMaxLoopsLimit(loopLimit);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L143

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1350-L1396

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Rewards/RewardsDistributor.sol

loopLimit_ is not checked before set _setMaxLoopsLimit. loopLimit_ can be zero value. Should be checked before set _setMaxLoopsLimit

function initialize(
        Comptroller comptroller_,
        IERC20Upgradeable rewardToken_,
        uint256 loopsLimit_,
        address accessControlManager_
    ) external initializer {
        comptroller = comptroller_;
        rewardToken = rewardToken_;
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager_);

        _setMaxLoopsLimit(loopsLimit_);
    }

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L111-L123

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L65-L77

##

## [G-10] Use nested if and, avoid multiple check combinations

- Instances(10)

- Approximate Gas Saved: 90 gas 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas 

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Comptroller.sol

755: if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {

FILE: 2023-05-venus/contracts/VToken.sol

837: if (redeemTokens == 0 && redeemAmount > 0) {

FILE: 2023-05-venus/contracts/Lens/PoolLens.sol

462: if (deltaBlocks > 0 && borrowSpeed > 0) {
483: if (deltaBlocks > 0 && supplySpeed > 0) {
506: if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {
526: if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {

FILE: 2023-05-venus/contracts/Rewards/RewardsDistributor.sol

348: if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {
386: if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
418: if (amount > 0 && amount <= rewardTokenRemaining) {
435: if (deltaBlocks > 0 && supplySpeed > 0) {
463: if (deltaBlocks > 0 && borrowSpeed > 0) {

```

##

## [G-11] No need to evaluate all expressions to know if one of them is true

> Instances(4) 

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
FILE: 2023-05-venus/contracts/Comptroller.sol

449:   if (skipLiquidityCheck || isDeprecated(VToken(vTokenBorrowed))) {

FILE: 2023-05-venus/contracts/VToken.sol

805: require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");

FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

164: require(
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );

364: require(
            (auction.startBlock == 0 && auction.status == AuctionStatus.NOT_STARTED) ||
                auction.status == AuctionStatus.ENDED,
            "auction is on-going"
        );

```
##

## [G-12] Amounts should be checked for 0 before calling a transfer

- Instances(7) 

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here

```solidity
FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol

183: erc20.safeTransfer(auction.highestBidder, previousBidAmount);
187: erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
229: erc20.safeTransfer(address(auction.markets[i]), bidAmount);
232: erc20.safeTransfer(address(auction.markets[i]), auction.marketDebt[auction.markets[i]]);
248: IERC20Upgradeable(convertibleBaseAsset).safeTransfer(auction.highestBidder, riskFundBidAmount);


```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL183C21-L184C1

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

416: token.safeTransferFrom(from, address(this), amount);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL416C9-L416C61


```solidity 
FILE: 2023-05-venus/contracts/RiskFund/RiskFund.sol

194: IERC20Upgradeable(convertibleBaseAsset).safeTransfer(shortfall, amount);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#LL194C9-L194C81

##

## [G-13] Use a more recent version of solidity

Upgrade to the latest solidity version 0.8.19 to get additional gas savings. See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

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

## [G-14] Non-usage of specific imports

CONTEXT:
ALL SCOPE CONTRACTS

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

##

## [G-15] Shorten the array rather than copying to a new one

- Instances (13)

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don’t have to be copied to a new, shorter array

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Lens/PoolLens.sol

126: VTokenBalances[] memory res = new VTokenBalances[](vTokenCount);
143: PoolData[] memory poolDataItems = new PoolData[](poolLength);
207: VTokenUnderlyingPrice[] memory res = new VTokenUnderlyingPrice[](vTokenCount);
228: RewardSummary[] memory rewardSummary = new RewardSummary[](rewardsDistributors.length);
256: BadDebt[] memory badDebts = new BadDebt[](markets.length);
390: VTokenMetadata[] memory res = new VTokenMetadata[](vTokenCount);
417: PendingReward[] memory pendingRewards = new PendingReward[](markets.length);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#LL126C9-L126C73

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Shortfall/Shortfall.sol

219: uint256[] memory marketsDebt = new uint256[](marketsCount);
386: uint256[] memory marketsDebt = new uint256[](marketsCount);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL219C6-L219C68

```solidity
FILE: Breadcrumbs2023-05-venus/contracts/Pool/PoolRegistry.sol

306: uint256[] memory newSupplyCaps = new uint256[](1);
307: uint256[] memory newBorrowCaps = new uint256[](1);
308: VToken[] memory vTokens = new VToken[](1);
355: VenusPool[] memory _pools = new VenusPool[](_numberOfPools);

```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#LL306C9-L308C51













