# Low Risk

## createPoolRegistry accepts any non-zero address for the price oracle

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L226](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L226)

```
require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle
address.");
```

The createPoolRegistry will accept any non-zero PriceOracle address and
create a pool without reverting. This is low impact as only admins
should be able to set Oracles, and an incorrect oracle can be corrected
later; however, to ensure that only valid Oracles are supplied to the
createRegistryPool function and any function calling
Comptroller.setPriceOracle, it is recommended to check the interface
function ‘getUnderlyingPrice’ either directly or using the ERC165
standard - see
[https://docs.openzeppelin.com/contracts/2.x/api/introspection#ERC165Checker](https://docs.openzeppelin.com/contracts/2.x/api/introspection#ERC165Checker)

—

## function \_transferIn assumes the ERC20 token is trusted, and will not manipulate the results of the balanceOf call. 

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L417](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L417)

If the input.asset value passed into the PoolRegistry/addMarket function
is malicious, it could return false values in the balanceOf function
that will result in more vTokens generated than what the actual balance
is after the transfer.

This is low likelihood as only authorised accounts can call the
addMarket function, but it’s recommended to include a check that the
token is known to the comptroller’s oracle as a safeguard against adding
dodgy tokens.

## \_swapAsset function can be private rather than internal as it’s not called anywhere else

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L230](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L230)

