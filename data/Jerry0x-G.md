[L201](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L201)Directly quote [vTokenAddress](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L187) without conversion.

[Incorrect comment](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L76)

You can remove [line 231](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L231) because [line 232](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L232) ensures that the value is greater than zero.