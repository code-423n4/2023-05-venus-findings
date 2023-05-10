### ****[G-01] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)****

There are two instances of this issuse.

- [https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL133C2-L141C1](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL133C2-L141C1)

```solidity
File: /contracts/VToken.sol
	133: function approve(address spender, uint256 amount) external override returns (bool) {
	134:        require(spender != address(0), "invalid spender address");
	135:
	136:       address src = msg.sender;
	137:        transferAllowances[src][spender] = amount;
	138:        emit Approval(src, spender, amount);
	139:        return true;
	140:    }
```

- **Mitigation**

```diff
133: function approve(address spender, uint256 amount) external override returns (bool) {
134:        require(spender != address(0), "invalid spender address");
135:
- 136:       address src = msg.sender;
- 137:        transferAllowances[src][spender] = amount;
- 138:        emit Approval(src, spender, amount);
+ 137:        transferAllowances[msg.sender][spender] = amount;
+ 138:        emit Approval(msg.sender, spender, amount);
139:        return true;
140:    }
```

- [https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL625C3-L635C6](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#LL625C3-L635C6)

```solidity
File: /contracts/VToken.sol
	625: function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
	626:        require(spender != address(0), "invalid spender address");
	627:
	628:        address src = msg.sender;
	629:        uint256 newAllowance = transferAllowances[src][spender];
	630:        newAllowance += addedValue;
	631:        transferAllowances[src][spender] = newAllowance;
	632:
	633:        emit Approval(src, spender, newAllowance);
	634:        return true;
	635:    }
```

- **Mitigation**

```diff
File: /contracts/VToken.sol
	625: function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
	626:        require(spender != address(0), "invalid spender address");
	627:
-	628:        address src = msg.sender;
+	629:        uint256 newAllowance = transferAllowances[msg.sender][spender];
	630:        newAllowance += addedValue;
+	631:        transferAllowances[msg.sender][spender] = newAllowance;
	632:
+	633:        emit Approval(msg.sender, spender, newAllowance);
	634:        return true;
	635:    }
```

## [G-02] `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`and `while`loops

`Note:` None of these findings were found by [bot findings - Gas](https://gist.github.com/CloudEllie/213965a3448230f5b615e7046f9dd26d)

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

There is one instance of this issuse.

```solidity
File: /contracts/Pool/PoolRegistry.sol
	399: _numberOfPools++;
```

- [https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L399](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L399)

## [G-03] Use modifiers instead of writing require statement.

`Note:` None of these findings were found by [bot findings - Gas](https://gist.github.com/CloudEllie/213965a3448230f5b615e7046f9dd26d)

see `@audit` tag

There is one instance of this issue.

```solidity
File: /contracts/VToken.sol
	525: require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens"); //@audit Use onlyOwner() Modifier
```

[https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L525](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L525)

- **Mitigation**

Use onlyOwner modifier

## [G‑04] Using `storage` instead of `memory` for structs/arrays saves gas

`Note:` None of these findings were found by [bot findings - Gas](https://gist.github.com/CloudEllie/213965a3448230f5b615e7046f9dd26d)

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

```solidity
File: /contracts/Comptroller.sol
687: VToken[] memory borrowMarkets = accountAssets[borrower];
```

- [https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L687](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L687)

## [G‑05] **The result of a function call should be cached rather than re-calling the function**

`Note:` None of these findings were found by [bot findings - Gas](https://gist.github.com/CloudEllie/213965a3448230f5b615e7046f9dd26d)

External calls are expensive. Consider caching the following: 

- `**RiskFund.sol._swapAsset()**` : ****Results of `underlyingAssetPrice` should be cached.**
    
    The result of function calls should be cached rather than re-calling the function
    
    `Note:` None of these findings were found by [bot findings - Gas](https://gist.github.com/CloudEllie/213965a3448230f5b615e7046f9dd26d)
    
    see `@audit` tag
    
    The instances below point to the second+ call of the function within a single function
    
    *There is one instance of this issue:*
    
    ```solidity
    240: uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
    241:            address(vToken) //@audit see Line 245 where underlyingAssetPrice calculated second time
    242:        );
    ```
    
    - [https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#LL240C8-L242C11](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#LL240C8-L242C11)