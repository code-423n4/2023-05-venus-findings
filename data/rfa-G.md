## [G-1] Using `calldata` pointer to declare array param

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154

Using calldata to store read-only array param might save gas


## [G-2] Unnecessary `vToken` variable declaration.

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L163

We might just pass `VToken(vTokens[i])` to `_addMarket` function without doing unnecessary MSTORE, especially in this case we are doing loop so it will save a lot by avoiding multiple MSTORE

RECOMMENDED MITIGATION STEP:

```solidity
	for (uint256 i; i < len; ++i) {
            // Remove line below
            // VToken vToken = VToken(vTokens[i]);

	    // Pass directly the value to this line
            _addToMarket(VToken(vTokens[i]), msg.sender);
            results[i] = NO_ERROR;

```


## [G-3] Using `unchecked{++i;}` for loop

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L162
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L217
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L274
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L304
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L376
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402
And more


