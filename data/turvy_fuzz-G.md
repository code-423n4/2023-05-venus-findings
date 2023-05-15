### Avoid unnecessary extra arithmetic operation on storage variable
line 236:  uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L236

as you can see the memory variable `balanceOfUnderlyingAsset` is set exactly to the value of `poolsAssetsReserves[comptroller][underlyingAsset]`; which never got modified and still the same in line 250 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L250, therefore it is a known tautology that in line 250, `poolsAssetsReserves[comptroller][underlyingAsset]` will always be equal to `0` i.e (x - x = 0). Therefore **poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset**; waste of gas.
Recommendation
So instead of performing unnecassary arithmetic that it's result will always be 0, simply set `poolsAssetsReserves[comptroller][underlyingAsset]` = `0` in line 250 and save the gas cost for the extra arithmetic operation

### convert same require checks to a single modifier
same checks in both functions:
**require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");**
**require(asset != address(0), "ReserveHelpers: Asset address invalid");**

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L39