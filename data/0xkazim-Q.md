# Findings Summary

| ID     | Title                                                              | Severity |
| ------ | -------------------------------------------------------------------| -------- |
| [L-01] | unncessary set of unnessasry exchange rate to memory               | low      |
| [L-02] |adding max Loop limit to the `preMintHook` function to avoid DOS if | low      |


# [G-01] unncessary set of unnessasry exchange rate to memory     

## Description

there is no need to set the exchange rate to memory in the `preMintHook` in the comptroller contract because we add it then into storage again. 
## context

```solidity

file: contract/shortfall.shorFall.sol

 function preMintHook(
        address vToken,
        address minter,
        uint256 mintAmount
    ) external override {
        _checkActionPauseState(vToken, Action.MINT);

        if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }

        uint256 supplyCap = supplyCaps[vToken];
        // Skipping the cap check for uncapped coins to save some gas
        if (supplyCap != type(uint256).max) {
            uint256 vTokenSupply = VToken(vToken).totalSupply();
            //@audit unnessasry exchange rate !
            Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
            uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);
            if (nextTotalSupply > supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
        }
        ...
```
## Recommendations

recommend to not set the `exchangeRate` into memory 



# [l-02] adding max Loop limit to the `preMintHook` function to avoid DOS if it have possibility to happen       

## Description

adding `_ensureMaxLoops` will be a good choice to the `preMintHook` function to avoid any DOS that may happen to the function where there is a lot of call into mints functions 

## context

 function preMintHook(
        address vToken,
        address minter,
        uint256 mintAmount
    ) external override {
        _checkActionPauseState(vToken, Action.MINT);

        if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }

        uint256 supplyCap = supplyCaps[vToken];
        // Skipping the cap check for uncapped coins to save some gas
        if (supplyCap != type(uint256).max) {
            uint256 vTokenSupply = VToken(vToken).totalSupply();
            Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
            uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);
            if (nextTotalSupply > supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
        }

        // Keep the flywheel moving
        uint256 rewardDistributorsCount = rewardsDistributors.length;
        //audit add max loop here !

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
        }
    }

## Recommendations
 add `_ensureMaxLoops` to this function to avoid DOS in some cases.