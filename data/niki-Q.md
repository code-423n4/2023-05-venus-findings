
### Do not use Deprecated Library Functions

#### Findings:
```
contracts\Pool\PoolRegistry.sol::322 => token.safeApprove(address(vToken), 0);
contracts\Pool\PoolRegistry.sol::323 => token.safeApprove(address(vToken), amountToSupply);
contracts\RiskFund\RiskFund.sol::258 => IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
contracts\RiskFund\RiskFund.sol::259 => IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);
```


