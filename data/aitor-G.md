## [GAS] Using `delete` statement can save gas

Setting a variable = 0 should cost 2900 gas by calling G_SReset.

Deleting the variable should call R_SCLEAR which adds 4800 gas to the refund counter.

Reference 1: https://solodit.xyz/issues/8917#

Reference 2: https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore

### Lines of code:

contract: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

`812: newMarket.collateralFactorMantissa = 0;`

`813: newMarket.liquidationThresholdMantissa = 0;`

`1353: snapshot.shortfall = 0;`

`1355: snapshot.liquidity = 0;`

contract: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

`424: accountBorrows[borrower].principal = 0;`

contract: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

`370: auction.highestBidBps = 0;`

`371: auction.highestBidBlock = 0;`

`376: auction.marketDebt[vToken] = 0;`

`410: remainingRiskFundBalance = 0;`

### Recomendation: use `delete` instead of `= 0`, for example:

`410: delete remainingRiskFundBalance;`