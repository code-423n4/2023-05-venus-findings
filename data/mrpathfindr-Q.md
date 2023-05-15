
Add a revert message instead of return;
---


Instances include:

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1250-L1252


https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1184-L1187

Recommendation:

Instead of using return to ignore/bypasss certain functionalities, it is better to revert the function with a message to increase the ability to debug issues. 

Change this:

```
 if (marketToJoin.accountMembership[borrower]) {
            // already joined
            return;
        }
```


to this 


```
 require (!marketToJoin.accountMembership[borrower], 'Already Joined' ) 
          
```
