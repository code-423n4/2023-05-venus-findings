[L-01] No possibility to list markets with underlying ERC-20 tokens not implementing decimals()
[https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L284](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L284)

## Vulnerability Details

When listing new asset (VToken) in an isolated pool in `PoolRegistry::addMarket()`, underlying token using function `decimals()`:

```
    function addMarket(AddMarketInput memory input) external {
        ...
        Comptroller comptroller = Comptroller(input.comptroller);
        uint256 underlyingDecimals = IERC20Metadata(input.asset).decimals();

```

The issue is that this is optional the [ERC-20](https://eips.ethereum.org/EIPS/eip-20) standard. Even though BSC officially uses [BEP-20](https://github.com/bnb-chain/BEPs/blob/master/BEP20.md#5113-decimals) standard, that implements this function, it's not guaranteed that the asset will have it, especially if it was copied from Ethereum. In this case, adding new market will always revert, making listing the market will not be possible.

## Impact

No possibility to list markets not implementing ERC20Metadata - `decimals()` function, leading to possible economic losses to the protocol

## Recommended Mitigation Steps

Consider adding underlying token decimals as an input parameter