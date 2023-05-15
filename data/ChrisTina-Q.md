## Inconsistent gap sizes

While the gap size can be set arbitrarily, it is often calculated so that the existing storage variables and the gap size add up to 50. This makes the gap size easier to calculate in future implementations.

From the [Openzeppelin documentation](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps):
```
The size of the __gap array is calculated so that the amount of storage used by a contract always adds up to the same number (in this case 50 storage slots).
```



This is not followed throughout the contracts, for example `VTokenInterface` has a gap size of 50 and doesn't account for the storage slots that are already taken up.

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/VTokenInterfaces.sol#L130

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/RiskFund/ReserveHelpers.sol#L26

## Comparison in `setProtocolSeizeShare` does not match documentation

The caller of `setProtocolSeizeShare` attempts to set the protocol share accumulated from liquidations, which according to the comments

```solidity
 * @dev must be less than liquidation incentive - 1
```

([source](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L303)). However, the check

```solidity
if (newProtocolSeizeShareMantissa_ + 1e18 > liquidationIncentive) {
    revert ProtocolSeizeShareTooBig();
}

```

will also go through if the new value is equal to the liquidation incentive - 1.

### Affected LOC

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L313-L315

### Recommendation

Compare if the new value is greater or equal

```solidity
if (newProtocolSeizeShareMantissa_ + 1e18 >= liquidationIncentive) {
    revert ProtocolSeizeShareTooBig();
}

```

or update the documentation to reflect that the value

```solidity
 * @dev must be less than or equal to liquidation incentive - 1
```

## Variable name shadows existing declaration

Consider renaming `address accessControlManager` to `address accessControlManager_` to differentiate between the function parameter and the global variable `accessControlManager` that already exists in the contract.

```solidity
    function initialize(uint256 loopLimit, address accessControlManager) external initializer {
        ...
        __AccessControlled_init_unchained(accessControlManager);
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L140

The same shadowed variable is present in `PoolRegistry.sol`
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L221
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L230

Likewise, the parameter `address owner` present in the following functions could be renamed to `address _owner`

```solidity
function balanceOfUnderlying(address owner) external override returns (uint256)
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L148-L151

```solidity
function allowance(address owner, address spender) external view override returns (uint256)
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L539-L540

```solidity
function balanceOf(address owner) external view override returns (uint256)
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L548-L549

## Misplaced comment

This comment in `PoolRegistry.sol` describes the code in line 236 and should be moved up there.

```solidity
 // Register the pool with this PoolRegistry
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L247

## Confusing documentation

The documentation states that

```solidity

@dev A borrow cap of -1 corresponds to unlimited borrowing.
```

which is confusing because the borrow cap is of type unsigned integer. I believe updating the documentation to reflect that the expected value for unlimited borrows is `type(uint256).max` would be clearer, as this is the value used to check if the borrow cap is unlimited.

```solidity
if (borrowCap != type(uint256).max)
```

([Source](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L351))

### Affected LOC

https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L829
https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L833
https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L855
https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L859
