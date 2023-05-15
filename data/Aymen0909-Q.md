# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Price returned by the oracle is not checked | Low | 3 |
| 2 | `incentiveBps` should have a maximum upper bound | Low | 1 |
| 3 | `shortfall` address not initialized in `RiskFund.sol` | Low | 1 |
| 4 | Wrong values emitted in the `RepayBorrow` event inside `VToken.healBorrow` | Low | 1 |


## Findings

### 1- Price returned by the oracle is not checked :

#### Risk : Low

There are many instances were the code does not check if the price (in usd) returned by the oracle is different from zero or not (the returned price can be zero if the oracle is unavailable), this can cause problems as the protocol functions will work with a wrong price in their calculations which will lead to major issues. And it's worth noting that the Comptroller contract does implement a `_safeGetUnderlyingPrice` function which is responsible for checking the price retuned from the oracle and revert if it is zero.

Instance of this issue :

File: PoolLens.sol [Line 266-268](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L266-L268)
```solidity
badDebt.badDebtUsd =
    VToken(address(markets[i])).badDebt() *
    priceOracle.getUnderlyingPrice(address(markets[i]));
```

File: RiskFund.sol [Line 240-242](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L240-L242)
```solidity
uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
    address(vToken)
);
```

File: Shortfall.sol [Line 393](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L393)
```solidity
uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
```

#### Mitigation
You should considere adding a non zero check on the price returned by the oracle.

### 2- `incentiveBps` should have a maximum upper bound  :

#### Risk : Low

The value of `incentiveBps` is in basis point of 10000 (meaning that 10000≈100%) but in the update function `updateIncentiveBps` there is no check to ensure that the fee is not above 10000, which means that the owner can set it to a large value (even greater than 10000).

Instance occurs in the code below :

File: Shortfall.sol [Line 307-313](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L307-L313)
```solidity
function updateIncentiveBps(uint256 _incentiveBps) external {
    _checkAccessAllowed("updateIncentiveBps(uint256)");
    require(_incentiveBps != 0, "incentiveBps must not be 0");
    uint256 oldIncentiveBps = incentiveBps;
    incentiveBps = _incentiveBps;
    emit IncentiveBpsUpdated(oldIncentiveBps, _incentiveBps);
}
```

#### Mitigation
You should considere adding an upper bound value when setting `incentiveBps`.

### 3- `shortfall` address not initialized in `RiskFund.sol` :

#### Risk : Low

The value of the variable `shortfall` is not set when the `RiskFund.sol` contract is initialized by the `initialize` function call and so the owner will be forced to call the `setShortfallContractAddress` later on to set the `shortfall` address, this can cause some problems for the protocol if the owner forgets to add the `shortfall` address after the initialization as the function `transferReserveForAuction` cannot be called without the `shortfall` address.

#### Mitigation
You should considere setting the `shortfall` address in the `initialize` function


### 4- Wrong values emitted in the `RepayBorrow` event inside `VToken.healBorrow` :

#### Risk : Low

In the `VToken.healBorrow` function there are two `RepayBorrow` events that can be emitted as it's shown in the code below :

File: VToken.sol [Line 399-422](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L399-L422)
```solidity
uint256 accountBorrowsPrev = _borrowBalanceStored(borrower);
uint256 totalBorrowsNew = totalBorrows;

uint256 actualRepayAmount;
if (repayAmount != 0) {
    // _doTransferIn reverts if anything goes wrong, since we can't be sure if side effects occurred.
    // We violate checks-effects-interactions here to account for tokens that take transfer fees
    actualRepayAmount = _doTransferIn(payer, repayAmount);
    totalBorrowsNew = totalBorrowsNew - actualRepayAmount;
    // @audit the first event
    emit RepayBorrow(payer, borrower, actualRepayAmount, 0, totalBorrowsNew);
}

// The transaction will fail if trying to repay too much
uint256 badDebtDelta = accountBorrowsPrev - actualRepayAmount;
if (badDebtDelta != 0) {
    uint256 badDebtOld = badDebt;
    uint256 badDebtNew = badDebtOld + badDebtDelta;
    totalBorrowsNew = totalBorrowsNew - badDebtDelta;
    badDebt = badDebtNew;

    // We treat healing as "repayment", where vToken is the payer
    // @audit the second event
    emit RepayBorrow(address(this), borrower, badDebtDelta, accountBorrowsPrev - badDebtDelta, totalBorrowsNew);
    emit BadDebtIncreased(borrower, badDebtDelta, badDebtOld, badDebtNew);
}
```

But for both events the 4th parameter which represent the account borrows is not correct :

* In the first event after the repayment of `actualRepayAmount` the borrower borrows balance should be equal to `accountBorrowsPrev - actualRepayAmount` but the value set in the event is 0.

* For the second event the borrower borrows balance after the repayment of bad debt should be equal to 0 but the value emitted is `accountBorrowsPrev - badDebtDelta` which is obviously wrong.

Those errors can be misleading to the users and external protocol who relies on accurate events data.

#### Mitigation

The correct values for the borrower borrows balance should be emitted in the `RepayBorrow` events as follows :


```solidity
uint256 accountBorrowsPrev = _borrowBalanceStored(borrower);
uint256 totalBorrowsNew = totalBorrows;

uint256 actualRepayAmount;
if (repayAmount != 0) {
    // _doTransferIn reverts if anything goes wrong, since we can't be sure if side effects occurred.
    // We violate checks-effects-interactions here to account for tokens that take transfer fees
    actualRepayAmount = _doTransferIn(payer, repayAmount);
    totalBorrowsNew = totalBorrowsNew - actualRepayAmount;
    emit RepayBorrow(payer, borrower, actualRepayAmount, accountBorrowsPrev - actualRepayAmount, totalBorrowsNew);
}

// The transaction will fail if trying to repay too much
uint256 badDebtDelta = accountBorrowsPrev - actualRepayAmount;
if (badDebtDelta != 0) {
    uint256 badDebtOld = badDebt;
    uint256 badDebtNew = badDebtOld + badDebtDelta;
    totalBorrowsNew = totalBorrowsNew - badDebtDelta;
    badDebt = badDebtNew;

    // We treat healing as "repayment", where vToken is the payer
    emit RepayBorrow(address(this), borrower, badDebtDelta, 0, totalBorrowsNew);
    emit BadDebtIncreased(borrower, badDebtDelta, badDebtOld, badDebtNew);
}
```