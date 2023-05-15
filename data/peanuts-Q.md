## [L-01] healAllowed hook is not available 

In VToken.sol#healBorrow, the comments state that 

```
     *   We assume that Comptroller does the seizing, so this function is only available to Comptroller.
     * @dev This function does not call any Comptroller hooks (like "healAllowed"), because we assume
     *   the Comptroller does all the necessary checks before calling this function.
```

However, the Comptroller has no hooks called healAllowed or any sort of checks to check whether the account is allowed to be healed. Setting as low as the function healAccount() can only be called by the Comptroller anyways. Would still be good to have a preHook function that checks if an account can be healed because sometimes, some account's debt may be far too underwater and healing the account back up to normal will accumulate a lot of bad debt.

```
    healBorrow(
        address payer,
        address borrower,
        uint256 repayAmount
    ) external override nonReentrant {
        if (msg.sender != address(comptroller)) {
            revert HealBorrowUnauthorized();
        }
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L578
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L390-L397