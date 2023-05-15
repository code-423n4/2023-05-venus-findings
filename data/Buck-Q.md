# This is a logic improvement proposal. 
## Moving one internal call deletes many lines of code and reduces risk of messing up when future functionality is implemented.  This removes the need for a function.
### function distributeBorrowerRewardToken() requires current data to execute and a call to function updateRewardTokenBorrowIndex() is required
### the update....() functions only call the internal _update functions
### the update....() functions are external and only called from Comptroller.sol
### the update....() functions are only called immediately before all distribute...() functions
### the distribute...() functions can call the internal function _update..() directly and remove significant overhead

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L163-L169
```
    function distributeBorrowerRewardToken(
        address vToken,
        address borrower,
        Exp memory marketBorrowIndex
    ) external onlyComptroller {
// add call to internal _update...Index
        _updateRewardTokenBorrowIndex(vToken, marketBorrowIndex);
        _distributeBorrowerRewardToken(vToken, borrower, marketBorrowIndex);
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#LL233-L235
```
    function distributeSupplierRewardToken(address vToken, address supplier) external onlyComptroller {
// add call to internal _update...Index
        _updateRewardTokenSupplyIndex(address(vToken));
        _distributeSupplierRewardToken(vToken, supplier);
    }
```
Now all the calls to the external update functions from Comptroller.sol can be consolidated.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L292-L308
preMintHook()  can remove call to updateRewardTokenSupplyIndex()
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L376-L379
preRedeemHook() can remove call to updateRewardTokenBorrowIndex()
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402-L406
prePayHook() can remove call to updateRewardTokenBorrowIndex()
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L521-L525
preSeizeHook() can remove call to updateRewardTokenSupplyIndex()
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L558-L562
preTransferHook() can remove call to updateRewardTokenSupplyIndex()

## Implementing this code reduces lines of code, adds readablity, and reduces the probability of adding more functionality and forgetting to call the update functions.