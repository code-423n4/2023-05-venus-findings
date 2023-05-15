### [G-01] Use != 0 instead of > 0 for unsigned integer comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.  

[contracts/Comptroller.sol#L367](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Comptroller.sol#L367)  
```
if (snapshot.shortfall > 0) {
```
There are 34 instances of this issue.

### [G-02] Long revert strings

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.  
If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.  

[contracts/Comptroller.sol#L692](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Comptroller.sol#L692)  
```
require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
```
There are 79 instances of this issue.

### [G-03] Don’t compare boolean expressions to boolean literals

Saves 9 gas per instance.  

[contracts/test/VTokenHarness.sol#L138](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/VTokenHarness.sol#L138)  
```
require(failTransferToAddresses[to] == false, "HARNESS_TOKEN_TRANSFER_OUT_FAILED");
```

### [G-04] Splitting require() statements that use && saves gas

Instead of a single require statement with multiple conditions, multiple require statements with one condition per statement should be used to save gas.  

[contracts/Comptroller.sol#L842](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Comptroller.sol#L842)  
```
require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");
```
There are 8 instances of this issue.
