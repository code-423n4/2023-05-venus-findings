## BNB chain halt
A blockchain halt, like the one that happened to Binance Smart Chain (BSC) in October 2022, is a significant event. When a blockchain halts, it means that no new blocks are being produced, so no transactions can be confirmed. It's akin to a system-wide freeze.

The impact on smart contracts in such a situation would be substantial. Here's what could potentially happen:

1. Transactions would be stalled: Any transaction that's broadcast during the halt wouldn't get confirmed until the network resumes. This includes transactions that interact with smart contracts. If a user attempted to interact with one of the provided contracts during the halt, their transaction would be stuck in a pending state.

2. [Auctions](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L158-L202) could be affected: In the given Auction contract, if the halt occurs during an ongoing auction, the auction would effectively be paused. No new bids could be placed, and the auction couldn't be finalized until the network resumes.

3. [Liquidations](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L292-L299) would be paused: In the case of the Liquidation contract, no liquidations could occur during the halt. If a position becomes undercollateralized during the halt, it couldn't be liquidated until the network resumes.

4. Time-based logic could be distorted: The block.timestamp is the timestamp when the block was mined, which should be roughly the current time. However, during a blockchain halt, no new blocks are mined, so the block.timestamp doesn't advance. This could distort the logic of contracts that rely on the block.timestamp.

5. Borrowers were unable to [repay](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L292-L299) their loans, this could indeed potentially lead to their positions becoming undercollateralized, especially if the halt lasted for a significant period of time and interest continued to accrue. If the protocol's interest calculation is such that interest continues to accrue during a halt, and the halt lasts long enough for positions to become undercollateralized, then those positions could indeed be at risk of liquidation once the network resumes.

Once the blockchain resumes, there could be a rush of transactions as users try to interact with contracts that were paused during the halt. This could lead to increased gas prices and potential network congestion.

As a mitigation strategy, contracts could implement a pause functionality that allows them to be manually paused and unpaused in case of such emergencies. However, this introduces a degree of centralization and needs to be handled with care. Furthermore, users and developers on the network should be prepared for such events and have contingency plans in place such as kicking in a grace period for borrowers to rectify their positions.

## Missing crucial validation input
In Shortfall.placeBid, if `auction.auctionType == AuctionType.LARGE_RISK_FUND` and the user accidentally entered 0 bidBps, no one else would be so dumb to place another 0 bid anymore although technically `bidBps <= auction.startBidBps` would make it still permissible.

One of the simplest and most effective ways to prevent this kind of issue is to include input validation in the placeBid function. You could add a requirement that bidBps must be greater than 0. This would prevent a 0 bid from being placed and effectively breaking the auction while making the careless bidder bear the entire debt for zero exchange of [riskFundBidAmount](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#LL244C13-L244C30).

Here is an example of how you could add this validation:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L158-L202

```diff
function placeBid(uint256 bidBps) public {
    // Input validation
+    require(bidBps > 0, "Bid must be greater than 0");

    // Existing logic
    ...
}
```
## Incorrect gap number
There are three (instead of two) state variables entailed in ReserveHelpers.sol. 

Consider having the gap number corrected refactored as follows:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L13-L26

```diff
    mapping(address => uint256) internal assetsReserves;

    // Store the asset's reserve per pool in the ProtocolShareReserve.
    // Comptroller(pool) -> Asset -> amount
    mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;

    // Address of pool registry contract
    address internal poolRegistry;

    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     */
-    uint256[48] private __gap;
+    uint256[47] private __gap;
```
## Hard coded NO_ERROR
In Comptroller._safeGetAccountSnapshot(), `err` matches with the hard coded constant `NO_ERROR` which equals `0`.

Consider removing the redundant check:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1402C14-L1417

```diff
    function _safeGetAccountSnapshot(VToken vToken, address user)
        internal
        view
        returns (
            uint256 vTokenBalance,
            uint256 borrowBalance,
            uint256 exchangeRateMantissa
        )
    {
        uint256 err;
        (err, vTokenBalance, borrowBalance, exchangeRateMantissa) = vToken.getAccountSnapshot(user);
-        if (err != 0) {
-            revert SnapshotError(address(vToken), user);
-        }
        return (vTokenBalance, borrowBalance, exchangeRateMantissa);
    }
```
## `maxLoopsLimit` could directionally make for loops prone to DoS  
[MaxLoopsLimitHelper._setMaxLoopsLimit()](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32) can only set a limit higher than the existing `maxLoopsLimit`. In the event `maxLoopsLimit` was accidentally overvalued, there would not be an option to downsize it. As a result, all functions dependent on [`_ensureMaxLoops()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L39C14-L43) could easily run out of gas in their loop iterations. Although a smaller array could always be inputted, it would disrupt calls having had to self gauge the supposed `maxLoopLimit`.

Consider flexibly making `_setMaxLoopsLimit()` to set a lower `maxLoopLimit` if need be by removing the require check:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32

```diff
    function _setMaxLoopsLimit(uint256 limit) internal {
-        require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

        uint256 oldMaxLoopsLimit = maxLoopsLimit;
        maxLoopsLimit = limit;

        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
    }
```
## Typo mistakes
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL385C48-L385C56

```diff
-     * @param borrower The account which would borrowed the asset
+     * @param borrower The account which would borrow the asset
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L808

```diff
-        require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure its really a VToken
+        require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure it is really a VToken
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#LL44C70-L44C86

```diff
-     * @dev Multiply an Exp by a scalar, truncate, then add an to an unsigned integer, returning an unsigned integer.
+     * @dev Multiply an Exp by a scalar, truncate, then add to an unsigned integer, returning an unsigned integer.
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#LL256C36-L256C43

```diff
-                        "RiskFund: finally path must be convertible base asset"
+                        "RiskFund: final path must be convertible base asset"
```
## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.13",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For example, the modifier instance below may be refactored as follows:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L33-L38

```diff
+    function _nonReentrant() private view {
+        require(_notEntered, "re-entered");
+        _notEntered = false;
+        _;
+        _notEntered = true; // get a gas-refund post-Istanbul
+    }

    modifier nonReentrant() {
-        require(_notEntered, "re-entered");
-        _notEntered = false;
-        _;
-        _notEntered = true; // get a gas-refund post-Istanbul
+        _nonReentrant();
        _;
    }
```
## Codes repeatedly called may be grouped into a modifier
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

```solidity
442:        if (!markets[vTokenCollateral].isListed) {
443:            revert MarketNotListed(address(vTokenCollateral));
444:        }

497:        if (!markets[vTokenCollateral].isListed) {
498:            revert MarketNotListed(vTokenCollateral);
499:        }
```
## Unneeded address cast
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L424-L444

```diff
    function preLiquidateHook(
        address vTokenBorrowed,
        address vTokenCollateral,
        address borrower,
        uint256 repayAmount,
        bool skipLiquidityCheck
    ) external override {
        // Pause Action.LIQUIDATE on BORROWED TOKEN to prevent liquidating it.
        // If we want to pause liquidating to vTokenCollateral, we should pause
        // Action.SEIZE on it
        _checkActionPauseState(vTokenBorrowed, Action.LIQUIDATE);

        oracle.updatePrice(vTokenBorrowed);
        oracle.updatePrice(vTokenCollateral);

        if (!markets[vTokenBorrowed].isListed) {
-            revert MarketNotListed(address(vTokenBorrowed));
+            revert MarketNotListed(vTokenBorrowed);
        }
        if (!markets[vTokenCollateral].isListed) {
-            revert MarketNotListed(address(vTokenCollateral));
+            revert MarketNotListed(vTokenCollateral);
        }
```

