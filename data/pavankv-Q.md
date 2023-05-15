## 1 . Add Zero address check :-

code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L187

## 2. Instead of return use revert :-
If the redeemer is not 'in' the market it should be revert because 
The REVERT instruction provides a way to stop execution and revert state changes, without consuming all provided gas and with the ability to return a reason.

code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1251


## 3 . [_checkActionPauseState()](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL1430C14-L1430C37) should be after check of market exist .:-

[_checkActionPauseState()](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL1430C14-L1430C37) function can be bypassed by calling non-exist market address and after _checkActionPauseState() function market exist check is there in some of the functions.Try to add [market exist check](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L333) first in function then you can check whether market is paused or not . This is valid finding because [_setActionPaused()](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1224) whcih add address to  [_actionPaused](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL1224C14-L1224C31) function add market address to mapping but it also in [markets](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#LL80C39-L80C46).

code snippent:-
In below code snippet add market exist check before check it's state

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L188
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L254
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L297
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L329
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L391
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L434
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L495
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L547
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1177


## 4.  


