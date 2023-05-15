1. 2 decimal/precision in baseUnit in ProtocolShareReserve is too little.

```solidity
    // Percentage of funds not sent to the RiskFund contract when the funds are released, following the project Tokenomics
    uint256 private constant protocolSharePercentage = 70;
    uint256 private constant baseUnit = 100;
```
Since the unit is used to calculate the ratio to send to protocolIncome / riskFund, right now with 2 decimals if the protocolIncome would have at worst, a floor down of 1%. It is advised to raise the decimals to at least 4-5 so the protocolIncome would not get impacted by division math. 

```solidity
        uint256 protocolIncomeAmount = mul_(
            Exp({ mantissa: amount }),
            div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
        ).mantissa;
```