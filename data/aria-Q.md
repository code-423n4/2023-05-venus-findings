# QA Report Venus

## [L - 0] Missing checks for`address(0x0)`when assigning values to`address`state variables

### Location -

In the PoolRegistry.sol contract in the initialize function at line 164.
[https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164)

### Code -

```solidity
function initialize(
        VTokenProxyFactory vTokenFactory_,
        JumpRateModelFactory jumpRateFactory_,
        WhitePaperInterestRateModelFactory whitePaperFactory_,
        Shortfall shortfall_,
        address payable protocolShareReserve_,
        address accessControlManager_
) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager_);

        vTokenFactory = vTokenFactory_;
        jumpRateFactory = jumpRateFactory_;
        whitePaperFactory = whitePaperFactory_;
        _setShortfallContract(shortfall_);
        _setProtocolShareReserve(protocolShareReserve_);
  }
```

### Explanation -

The variables `whitePaperFactory_, jumpRateFactory_, vTokenFactory_` might be `address(0x0)`, there should  be a check to make sure they are different from `address(0x0)` before initializing storage values. 

## [L - 1] Constant initialised to a different value then intended

### Location -

In the `WhitePaperInterestRateModel.sol` contract at line 17.
[https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/WhitePaperInterestRateModel.sol#L17](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/WhitePaperInterestRateModel.sol#L17)

### Code -

```solidity
/**
 * @notice The approximate number of blocks per year that is assumed by the interest rate model
 */
 uint256 public constant blocksPerYear = 2102400;
```

### Explanation -

The value `blocksPerYear` is defined to be 2102400 is different from the actual blocks per year in the BNB chain. We can calculate the number of seconds in a year to be `365*24*60*60 = 31536000`. And by dividing this by 2102400 we will get that there is a block every 15 seconds. This assumption is wrong since on the bnb chain a block is created every 3 seconds.
We can also see the correct value 1051200 in the `BaseJumpRateModelV2.sol` for further confirmation. 

## [L - 2]  User can front run transaction to stop liquidation.

### Location -

In the `Comptroller.sol` contract in function `liquidateAccount` at line 692.
[https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L692](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L692)

### Code -

```solidity
for (uint256 i; i < marketsCount; ++i) {
        (, uint256 borrowBalance, ) = _safeGetAccountSnapshot(borrowMarkets[i], borrower);
        require(borrowBalance == 0, "Nonzero borrow balance after liquidation");
    }
```

### Explanation -

In some cases, in this system some asset might be liquidated,this function gives us the ability to liquidate a “bad position”. At the end of this function, there is a condition to make sure the process has worked and that the user has no assets left. This means that in cases where the user’s complete balance wasn’t removed his assets won’t be liquidated. So if a user wants to avoid liquidation he can change his assets in such a way that the transaction and orders for the liquidation won’t clear all of his assets and the transaction will not liquidate his assets. This can be done by changing balances taking different loans and entering markets by the user. If the user will have the power to change those values and front run the transaction consistently he will be able to avoid liquidation which would be a big vulnerability.  This will also hurt some users since front running will make their transaction revert and the gas will be wasted.

## [N - 0] typos

### Location -

In the `PoolRegistry.sol` contract at line 121.
[https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Pool/PoolRegistry.sol#L211](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Pool/PoolRegistry.sol#L211)

### Code -

```solidity
* @return proxyAddress The the Comptroller proxy address
```

### Explanation -

The is written twice.

## [N - 1] consider adding functions for single supply cap and single borrowCap.

### Location -

In the `PoolRegistry.sol` contract in the `addMarket` function at line 310.
[https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Pool/PoolRegistry.sol#L310](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Pool/PoolRegistry.sol#L310)

### Code -

```solidity
newSupplyCaps[0] = input.supplyCap;
newBorrowCaps[0] = input.borrowCap;
vTokens[0] = vToken;

comptroller.setMarketSupplyCaps(vTokens, newSupplyCaps);
comptroller.setMarketBorrowCaps(vTokens, newBorrowCaps);
```

### Explanation -

In this case we use the functions `setMarketBorrowCaps` and `setMarketSupplyCaps` when a single value is in the `newSupplyCaps`, `newBorrowCaps` arrays. Consider implementing another function for this case where there is only one value.