###On VToken.sol.
 you can save more gas by changing line https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1129
> accountTokens[borrower] = accountTokens[borrower] - seizeTokens;
 to 
> accountTokens[borrower] -= seizeTokens;
