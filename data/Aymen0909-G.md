# Gas Optimizations

## Summary

|       | Issue        | Instances    |
| :---: |:-------------|:------------:|
| 1  | State variables should be packed to save storage slots | 2 |
| 2  | Variables inside struct should be packed to save gas | 1 |
| 3  | Using `storage` instead of `memory` for struct/array saves gas | 5 |
| 4  | `storage` variable should be cached into `memory` variables instead of re-reading them | 14 |
| 5  | Emit events outside the for loops | 2 |
| 6  | Use `unchecked` blocks to save gas | 4 |
| 7  | Use `calldata` instead of `memory` for function parameters type | 7 |
| 8  | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 5 |
| 9  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | 6 |
| 10 | `memory` values should be emitted in events instead of `storage` ones |  2 |
| 11 | Using `delete` statement can save gas | 4 |


## Findings

### 1- State variables should be packed to save storage slots :

State variables inside contract should be packed if possible to save storage slots and thus will save gas cost.

There are 2 instances of this issue:

* Instance 1 :

```
File: Pool/PoolRegistry.sol

78      address payable public protocolShareReserve;
93      uint256 private _numberOfPools;
```

As the variable `_numberOfPools` represente the number of pool created its value can't realistically overflow 2^96, so it should be packed with the address variable `protocolShareReserve` (which only takes 20 bytes), this will **save 1 storage slots** and thus saves a lot of gas, the code should be modified as follow :

```
78      address payable public protocolShareReserve;
93      uint96 private _numberOfPools;
```

* Instance 2 :

```
File: Shortfall/Shortfall.sol

78      uint256 private incentiveBps;
63      uint256 public nextBidderBlockLimit;
66      uint256 public waitForFirstBidder;
69      address public convertibleBaseAsset;
```

As the variables `nextBidderBlockLimit` and `waitForFirstBidder` represents block numbers their values can't realistically overflow 2^32, and because the variable `incentiveBps` can't be greater than 10000 its value also can't overflow 2^32, so all those variables should be packed with the address variable `convertibleBaseAsset` (which only takes 20 bytes), this will **save 3 storage slots** and thus saves a lot of gas, the code should be modified as follow :

```
address public convertibleBaseAsset;
uint32 private incentiveBps;
uint32 public nextBidderBlockLimit;
uint32 public waitForFirstBidder;  
```

### 2- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance of this issue:

**File: Pool/PoolRegistryInterface.sol** [Line 13-28](https://github.com/code-423n4/2022-10-blur/blob/main/contracts/lib/OrderStructs.sol#L13-L28)

```
struct VenusPool {
    string name;
    address creator;
    address comptroller;
    uint256 blockPosted;
    uint256 timestampPosted;
}
```

As the variables `blockPosted` and `timestampPosted` represents the number of pssed blocks and the timestamp of the pool they can't realistically overflow 2^48 so their values should be packed with the address variable `comptroller` (which only takes 20 bytes), this will **save 2 storage slots** and thus saves a lot of gas, the struct should be modified as follow :

```
struct VenusPool {
    string name;
    address creator;
    address comptroller;
    uint48 blockPosted;
    uint48 timestampPosted;
}
```

### 3- Using `storage` instead of `memory` for struct/array saves gas :

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 

The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

There are 5 instances of this issue :

**File: PoolRegistry.sol** [Line 394](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L394)
```
VenusPool memory venusPool = _poolByComptroller[comptroller];
```

**File: Comptroller.sol** [Line 213](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L213)
```
VToken[] memory userAssetList = accountAssets[msg.sender];
```

**File: Comptroller.sol** [Line 579](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L579)
```
VToken[] memory userAssets = accountAssets[user];
```

**File: Comptroller.sol** [Line 687](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L687)
```
VToken[] memory borrowMarkets = accountAssets[borrower];
```

**File: Comptroller.sol** [Line 1304](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1304)
```
VToken[] memory assets = accountAssets[account];
```

### 4- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 14 instances of this issue :

**File: PoolRegistry.sol** [Line 355-356](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L355-L356)
```
VenusPool[] memory _pools = new VenusPool[](_numberOfPools);
for (uint256 i = 1; i <= _numberOfPools; ++i)
```

In the code linked above the value of `_numberOfPools` is read multiple times (inside the for loop) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: PoolRegistry.sol** [Line 403-407](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L403-L407)
```
_poolsByID[_numberOfPools] = comptroller;
_poolByComptroller[comptroller] = pool;

emit PoolRegistered(comptroller, pool);
return _numberOfPools;
```

In the code linked above the value of `_numberOfPools` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 166-190](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L166-L190)

In the code linked above the value of `auction.highestBidder` is read multiple times (also inside the for loop) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 166-181](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L166-L181)

In the code linked above the value of `auction.highestBidBps` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 165-179](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L165-L179)

In the code linked above the value of `auction.auctionType` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 181-193](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L181-L193)

In the code linked above the value of `auction.marketDebt[auction.markets[i]]` is read multiple times (4 times inside a for loop) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 214-253](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L214-L253)

In the code linked above the value of `auction.highestBidder` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 224-236](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L224-L236)

In the code linked above the value of `auction.markets[i]` is read multiple times (7 times inside the for loop) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 227-241](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L227-L241)

In the code linked above the value of `auction.auctionType` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 228-233](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L228-L233)

In the code linked above the value of `auction.marketDebt[auction.markets[i]]` is read multiple times (3 times inside the for loop) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Shortfall.sol** [Line 228-254](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L228-L254)

In the code linked above the value of `auction.highestBidBps` is read multiple times (also inside the for loop) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


**File: MaxLoopsLimitHelper.sol** [Line 40-41](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L40-L41)

In the code linked above the value of `maxLoopsLimit` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: RiskFund.sol** [Line 191-194](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L191-L194)

In the code linked above the value of `shortfall` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: RiskFund.sol** [Line 258-260](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L258-L260)

In the code linked above the value of `pancakeSwapRouter` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 5- Emit events outside the for loops :

Putting an event inside a for loop means that the event will be emitted at each iteration which will cost a lot of gas in the end, you should consider emitting a global event outside the for loop when possible.

There are 2 instances of this issue :

**File: Comptroller.sol** [Line 848](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L848)
```
emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);
```

The event in the code linked above is emitted at each iteration which can be avoided by emitting a final event after the for loop containing all the vTokens and the correspanding newBorrowCaps as follows :

```
emit NewBorrowCap(vTokens, newBorrowCaps);
```

**File: Comptroller.sol** [Line 873](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L873)
```
emit NewSupplyCap(vTokens[i], newSupplyCaps[i]);
```

The event in the code linked above is emitted at each iteration which can be avoided by emitting a final event after the for loop containing all the vTokens and the correspanding newSupplyCaps as follows :

```
emit NewSupplyCap(vTokens, newSupplyCaps);
```


### 6- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 4 instances of this issue:

**File: ProtocolShareReserve.sol** [Line 75](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L75)
```
poolsAssetsReserves[comptroller][asset] -= amount;
```

In the code linked above the operation cannot underflow because of the check in line [72](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L72) ensures that we always have `amount > amountToDecrement`, so the operation should be marked as `unchecked` to save gas.


**File: Shortfall.sol** [Line 415](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L415)
```
remainingRiskFundBalance = remainingRiskFundBalance - maxSeizeableRiskFundBalance;
```

In the code linked above the operation cannot underflow because of the check in line [406](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L406), so the operation should be marked as `unchecked` to save gas.

**File: VToken.sol** [Line 493](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L493)
```
uint256 badDebtNew = badDebtOld - recoveredAmount_;
```

In the code linked above the operation cannot underflow because of the check in line [490](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L490), so the operation should be marked as `unchecked` to save gas.


**File: VToken.sol** [Line 1223](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1223)
```
totalReservesNew = totalReserves - reduceAmount;
```

In the code linked above the operation cannot underflow because of the check in line [1215](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1215), so the operation should be marked as `unchecked` to save gas.


### 7- Use `calldata` instead of `memory` for function parameters type :

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

There are 7 instances of this issue:

```
File: Rewards/RewardsDistributor.sol

197     function setRewardTokenSpeeds(
            VToken[] memory vTokens,
            uint256[] memory supplySpeeds,
            uint256[] memory borrowSpeeds
        )
277     function claimRewardToken(address holder, VToken[] memory vTokens)
671     function _updateBucketExchangeRates(
            address pool_,
            uint256[] memory indexes_
        )

File: Lens/PoolLens.sol

388     function vTokenMetadataAll(VToken[] memory vTokens)
412     function _calculateNotDistributedAwards(
            address account,
            VToken[] memory markets,
            RewardsDistributor rewardsDistributor
        )

File: Comptroller.sol

154     function enterMarkets(address[] memory vTokens)
412     function _calculateNotDistributedAwards(
            address account,
            VToken[] memory markets,
            RewardsDistributor rewardsDistributor
        )
```

### 8- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 5 instances of this issue :

**File: Pool/PoolRegistry.sol**
```
83:     mapping(address => VenusPoolMetaData) public metadata;
98:     mapping(address => VenusPool) private _poolByComptroller;
103:    mapping(address => mapping(address => address)) private _vTokens;
```

These mappings could be refactored into the following struct and mapping for example (this also enables to pack variables inside the struct to save gas) :

```
struct ComptrollerInfo {
    VenusPoolMetaData metadata;
    VenusPool _poolByComptroller;
    mapping(address => address) _vTokens;
}
    
mapping(address => ComptrollerInfo) public comptrollerMap;
```


**File: RiskFund/ReserveHelpers.sol**
```
13:     mapping(address => uint256) internal assetsReserves;
17:     mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;
```

These mappings could be refactored into the following struct and mapping for example (this also enables to pack variables inside the struct to save gas) :

```
struct ReserveInfo {
    uint256 assetsReserves;
    mapping(address => uint256) poolsAssetsReserves;
}
    
mapping(address => ReserveInfo) internal reserves;
```


### 9- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 6 instances of this issue:

```
File: RiskFund/ProtocolShareReserve.sol

74      assetsReserves[asset] -= amount;
75      poolsAssetsReserves[comptroller][asset] -= amount;votesUsed);

File: RiskFund/ReserveHelpers.sol

66      assetsReserves[asset] += balanceDifference;
67      poolsAssetsReserves[comptroller][asset] += balanceDifference;

File: RiskFund/RiskFund.sol

249     assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
250     poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;
```


### 10- `memory` values should be emitted in events instead of `storage` ones :

The values emitted in events shouldn’t be read from storage but the existing memory values should be used instead, this will save **~101 GAS**.

There are 2 instances of this issue :

**File: RewardsDistributor.sol** [Line 268](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L268)
```
emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
```

The memory value `contributorAccrued` should be emitted as follow :

```
emit ContributorRewardsUpdated(contributor, contributorAccrued);
```

**File: MaxLoopsLimitHelper.sol** [Line 31](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L31)
```
emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
```

The memory value `limit` should be emitted as follow :

```
emit ContributorRewardsUpdated(contributor, limit);
```


### 11- Using `delete` statement can save gas :

When reseting a variable to its default value (0 for uint or false for boolean) it's cost less gas to use the `delete` operator rather than assigning the default value to the variable.

There are 4 instances of this issue :

**File: Shortfall.sol** [Line 370-371](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L370-L371)
```
auction.highestBidBps = 0;
auction.highestBidBlock = 0;
```

**File: Shortfall.sol** [Line 376](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L376)
```
auction.marketDebt[vToken] = 0;
```

**File: VToken.sol** [Line 424](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L424)
```
accountBorrows[borrower].principal = 0;
```