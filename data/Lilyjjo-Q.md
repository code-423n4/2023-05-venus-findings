## [NC-1] RiskFund.sol’s `swapPoolAssets()` failures can be isolated

In RiskFund.sol’s function `swapPoolAssets()`, an approved party can submit a list of markets to perform a swap conversion to the target base asset. Currently this function will fail if ANY of the submitted market swap fail. This is suboptimal as there are many points of failure in the `swapPoolsAssets()` logic for each market. A different function structure would allow successful swaps to succeed while recording the failure of bad swaps, decreasing gas waste. 

Consider the scenario of approved party P submitting the list of [vTokenA, vTokenB, vTokenC] to convert into the desired base asset.

Let vTokenA and vTokenB successfully swap but let vTokenC fail due to either illogical input path or due to market conditions making the swap fail to meet the required minimum out. With the current implementation, the successful swaps for vTokenA and vTokenB will be reverted due to vTokenC’s failure. This is suboptimal and can be avoided by using a try-catch structure to isolate failures.

Consider the below pseudo code’s structure as an example. The function isolates the likely to fail code’s possible failure while (1) recording the failure it if it does happen and (2) allowing the non-failing code to still succeed. 

```solidity
function swapPoolsAssets(
        address[] calldata markets, // vTokens
        uint256[] calldata amountsOutMin,
        address[][] calldata paths
    ) external override returns (uint256) {
        _checkAccessAllowed("swapPoolsAssets(address[],uint256[],address[][])");
        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");
        require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");
        require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");

        uint256 totalAmount;
        uint256 marketsCount = markets.length;

        _ensureMaxLoops(marketsCount);

        for (uint256 i; i < marketsCount; ++i) {
            VToken vToken = VToken(markets[i]);
            address comptroller = address(vToken.comptroller());

            PoolRegistry.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
            require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");
            require(Comptroller(comptroller).isMarketListed(vToken), "market is not listed");
						
            // Main change below: using try-catch for failure prone logic
            try this._swapAsset(vToken, comptroller, amountsOutMin[i], paths[i]) returns (uint256 swappedTokens) {
                // if successful, keep original logic
                poolReserves[comptroller] = poolReserves[comptroller] + swappedTokens;
                totalAmount = totalAmount + swappedTokens;
            } catch (bytes memory reason) {
                // if failure, record why but don't fail whole loop
                emit Log(market, reason);
            }
        }

        emit SwappedPoolsAssets(markets, amountsOutMin, totalAmount);

        return totalAmount;
    }
```

### Remediations to Consider:

- Using a try-catch for the prone-to-failure logic in order to enable isolated and successful logic to succeed.