1.Split && in require save more gas
RewardsDistributor.sol#L204#L207
Comptroller.sol#L842
VToken.sol#L1365

```solidity
require(
    numTokens == supplySpeeds.length && numTokens == borrowSpeeds.length,
    "RewardsDistributor::setRewardTokenSpeeds invalid input"
);
```

2.Put loop ++i into unchecked to save more gas
RewardsDistributor.sol#L209
RewardsDistributor.sol#L282
Comptroller.sol#L162
Comptroller.sol#L271
Comptroller.sol#L274
Comptroller.sol#L304
Comptroller.sol#L376
Comptroller.sol#L402
Comptroller.sol#L521
Comptroller.sol#L558
Comptroller.sol#L584
Comptroller.sol#L611
Comptroller.sol#L669
Comptroller.sol#L690
Comptroller.sol#L819
Comptroller.sol#L846
Comptroller.sol#L876
Comptroller.sol#L932
Comptroller.sol#L948
Comptroller.sol#L1122
Comptroller.sol#L1028
Comptroller.sol#L1307
PoolLens.sol#L127
PoolLens.sol#L145
PoolLens.sol#L208
PoolLens.sol#L229
PoolLens.sol#L263
PoolLens.sol#L391
PoolLens.sol#L418
Shortfall.sol#L175
Shortfall.sol#L223
Shortfall.sol#L374
Shortfall.sol#L389

3.Not necessary to set `msg.sender` to local variable.
Comptroller.sol#L582
VToken.sol#L136
VToken.sol#L628
VToken.sol#L648

4.Read local variable `newCloseFactorMantissa` instead of storage variable `closeFactorMantissa` to save more gas in emit
Comptroller.sol#L709

```solidity
    function setCloseFactor(uint256 newCloseFactorMantissa) external {
    _checkAccessAllowed("setCloseFactor(uint256)");
    require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");
    require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");

    uint256 oldCloseFactorMantissa = closeFactorMantissa;
    closeFactorMantissa = newCloseFactorMantissa;
-   emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);
+   emit NewCloseFactor(oldCloseFactorMantissa, newCloseFactorMantissa);
    }

```

5.Constructors can be marked payable
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.
Comptroller.sol#L127
RewardsDistributor.sol#L104
VToken.sol#L41
PoolRegistry.sol#L150

6.Use delete instead to set storage variable to 0
Comptroller.sol#L812#L813

7.Not necessary to set a local variable when you can return the storage variable directly.
Comptroller.sol#L1058#L1062

```solidity
    function getAssetsIn(address account) external view returns (VToken[] memory) {
-    VToken[] memory assetsIn = accountAssets[account]; //@audit need a local variable ?

-    return assetsIn;
+    return accountAssets[account];
    }

```

8.Read local variable instead of read storage variable to save more gas.
MaxLoopsLimitHelper.sol#L31

```solidity
    function _setMaxLoopsLimit(uint256 limit) internal {
    require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

    uint256 oldMaxLoopsLimit = maxLoopsLimit;
    maxLoopsLimit = limit;

-   emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
+   emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, limit);
    }

```

9.The access to the `maxLoopsLimit` is performed within the `_ensureMaxLoops` function check, so it may seem unnecessary to have it set as public. Changing it to private could save more gas
MaxLoopsLimitHelper.sol#L6

10.In the `_addMarket` function, incrementing the variable by 1 using ++i is more gas-efficient than accessing the length of the storage variable, since the length is increased by 1. This is because accessing the length of an array in storage involves a storage read operation, which is more expensive than modifying a variable in memory
Comptroller.sol#L1205#L1216

```solidity
  function _addMarket(address vToken) internal {
    uint256 marketsCount = allMarkets.length;

    for (uint256 i; i < marketsCount; ++i) {
      if (allMarkets[i] == VToken(vToken)) {
        revert MarketAlreadyListed(vToken);
      }
    }
    allMarkets.push(VToken(vToken));
-   marketsCount = allMarkets.length;
+   unchecked{++marketsCount};
    _ensureMaxLoops(marketsCount);
  }

```

11.<array>.length `markets.length` Should Not Be Looked Up In Every Loop Of A For-loop
PoolLens.sol#L263
PoolLens.sol#L418

12.Use local variable cache instead of read storage variable mutile times in for loop.
PoolRegistry.sol#L354#L361

```solidity
  function getAllPools() external view override returns (VenusPool[] memory) {
+   uint256 numberOfPools = _numberOfPools;
-   VenusPool[] memory _pools = new VenusPool[](_numberOfPools);
+   VenusPool[] memory _pools = new VenusPool[](numberOfPools);
-   for (uint256 i = 1; i <= _numberOfPools; ++i) {
+   for (uint256 i = 1; i <= numberOfPools; ++i) {
      address comptroller = _poolsByID[i];
      _pools[i - 1] = (_poolByComptroller[comptroller]);
    }
    return _pools;
  }

```

13.Use bytes32 instead of string save more gas in solidity struct.
PoolLens.sol#L16#L30