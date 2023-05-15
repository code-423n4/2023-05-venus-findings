# Summary
|ID     | Finding|  Gas saved per instance | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Use block.number directly| 15 |  12 |
|G-02      |`_checkActionPauseState()` can be more efficiënt| 70 |  1 |
|G-03      |Don't assign global variables to a memory variable| 15 |  10 |
|G-04      |Don't initialize variables inside a loop| 40 |  - |
|G-05      |Use do-while loop| 150 |  - |
|G-06      |Sort operations using short-circuit mode| - |  - |
|G-07      |Don't emit events in a for loop| 3000 |  2 |
|G-08      |Emitting events can be more optimized| - |  11 |
|G-09      |uint48 is more than enough for block numbers| - |  3 |
# Details
## [G-01] Use block.number directly
The [VToken](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1430-L1432) contract has an internal view function to get the current block.number. However block.number is a global variable and only costs 2 gas. Which is cheaper than calling the internal view function every time. This change won't lower readibility because the function is almost similar to block.number.
## [G-02] `_checkActionPauseState()` can be more efficiënt
The function [`_checkActionPauseState()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1430-L1434) is used in tons of functions of the Comptroller contract. So it's important to make it as efficient as possible.

The function calls `actionPaused()` to see if a certain action is paused. `actionPaused()` checks the _actionPaused mapping and returns a boolean. However it's much more efficient to directly check the mapping. Around 70 gas can be saved for every function that uses `_checkActionPauseState()`.

```diff
    function _checkActionPauseState(address market, Action action) private view {
-       if (actionPaused(market, action)) {
+       if (_actionPaused[market][action]) {
            revert ActionPaused(market, action);
        }
    }
```

## [G-03] Don't assign global variables to a memory variable
Block and transaction properties like msg.sender only costs 2 gas to obtain. Reading and saving to memory both cost 3 gas. With all the additional stack operations this will result in quite a bit more gas than using msg.sender instead of the memory variable.

This optimization can be done in the VToken functions [`increaseAllowance()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L625-L635) and [`decreaseAllowance()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L645-L660). It will save around 15 gas for both of these functions.

```diff
L625
    function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
        require(spender != address(0), "invalid spender address");

-       address src = msg.sender;
-       uint256 newAllowance = transferAllowances[src][spender];
+       uint256 newAllowance = transferAllowances[msg.sender][spender];
        newAllowance += addedValue;
-       transferAllowances[src][spender] = newAllowance;
+       transferAllowances[msg.sender][spender] = newAllowance;

-       emit Approval(src, spender, newAllowance);
+       emit Approval(msg.sender, spender, newAllowance);
        return true;
    }
```
## [G-04] Don't declare variables inside a loop
If a memory variable is only used inside the for loop then it's cheaper to declare a variable before the loop starts. The longer the for loop, the more gas will be saved. 
For a loop of 10 iterations, around 40 gas will be saved.
```diff
+       Vtoken vtoken;
        for (uint256 i; i < len; ++i) {
-           VToken vToken = VToken(vTokens[i]);
+           vToken = VToken(vTokens[i]);

            _addToMarket(vToken, msg.sender);
            results[i] = NO_ERROR;
        }
```

## [G-05] Use do-while loop
Using a do-while instead of a for loop is more gas efficiënt. However this can only done if the length of the array is more than 0. Otherwise it will run the first iteration while it doesn't do it in a for loop. A do-while saves around 150 gas for 10 iterations. The more iterations the more gas it will save.

A do-while loop can be coded like the following:
```solidity
    for (uint256 i; i < len; ++i) {
        VToken vToken = VToken(vTokens[i]);

        _addToMarket(vToken, msg.sender);
        results[i] = NO_ERROR;
    }
```
Can be turned into:
```solidity
    do {
        VToken vToken = VToken(vTokens[i]);

        _addToMarket(vToken, msg.sender);
        results[i] = NO_ERROR;
        ++i;
    } while(i<len);
```

## [G-06] Sort operations using short-circuit rules
The operators `&&` and `||` are both logical operators, this mean they use the short-circuiting rules. For `&&` this means, when the first operand returns false, the second one will not be evaluated. `||` is the opposite, if the first operand returns true, the second one will not be evaluated.

With this in mind, gas can be saved when you put the value that will likely be the most true `||` as the first operand. And the opposite for `&&`.

## [G-07] Don't emit events in a for loop
Emitting events is a gas expensive operation. Emitting an event can cost anywhere between 1000 and 3000 gas depending on how many and the size of arguments you provide.
This is why it should never be done in a for loop. Instead emit a whole array in one event. Again, the longer the loop the more gas will be saved.

[Comptroller.sol#L836-L850](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L836-L850): 3000 gas saved
```diff
L58:
-   event NewBorrowCap(VToken indexed vToken, uint256 newBorrowCap);
+   event NewBorrowCap(VToken[] indexed vToken, uint256[] newBorrowCap);
    function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {
        _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");

        uint256 numMarkets = vTokens.length;
        uint256 numBorrowCaps = newBorrowCaps.length;

        require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");

        _ensureMaxLoops(numMarkets);

        for (uint256 i; i < numMarkets; ++i) {
            borrowCaps[address(vTokens[i])] = newBorrowCaps[i];
-           emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);
        }
+       emit NewBorrowCap(vTokens, newBorrowCaps);
    }
```
Same thing can be done at:
- [Comptroller.sol#L873](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L873)
## [G-08] Emitting events can be more optimized
This is mostly used when an owner changes a value. Then the old and the new value are arguments for the changed event.
However the old value is always saved to memory first so it can later be used in the event. This is not necessary. 
The event can be emitted before the new value gets changed and hereby saving some memory and stack operations.

[Comptroller.sol#L836-L850](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L836-L850)
```solidity
    function setPriceOracle(PriceOracle newOracle) external onlyOwner {
        require(address(newOracle) != address(0), "invalid price oracle address");

        PriceOracle oldOracle = oracle;
        oracle = newOracle;
        emit NewPriceOracle(oldOracle, newOracle);
    }
```
Can be changed to
```solidity
    function setPriceOracle(PriceOracle newOracle) external onlyOwner {
        require(address(newOracle) != address(0), "invalid price oracle address");

        emit NewPriceOracle(oracle, newOracle);
        oracle = newOracle;
    }
```
This optimization can also be made at:
- [Comptroller.sol#L709](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L709)
- [Comptroller.sol#L791](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L791)
- [Comptroller.sol#L917](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L917)
- [VToken.sol#L319](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L319)
- [VToken.sol#L1262](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1262)
- [PoolRegistry.sol#L337](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L337)
- [PoolRegistry.sol#L347](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L347)
- [PoolRegistry.sol#L427](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L427)
- [PoolRegistry.sol#L436](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L436)
## [G-09] uint48 is more than enough for block numbers
Even with 10000 blocks per second, uint48 will be enough until the year 2861. Because of this some structs can be packed into a fewer storage slots. The same thing is also true for timestamps, the maximum value of uint48 is almost year 9 million. 

The struct [`Auction`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L34-L45) has 2 block values: `startBlock` and `highestBidBlock`. Both of these are uint256. When you lower these to uint48, at least 1 extra storage slot will be saved.

Also used in:
- [`PoolData and VTokenMetadata`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L16-L52)
- [`VenusPool`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistryInterface.sol#L8-L14)
