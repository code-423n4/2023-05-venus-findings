## 1 . Add Zero address check :-

code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L187

## 2. Instead of return use revert :-
If the redeemer is not 'in' the market it should be revert because 
The REVERT instruction provides a way to stop execution and revert state changes, without consuming all provided gas and with the ability to return a reason.

code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1251