# Gas Saving

## Structs - type ordering

> [https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L34](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L34)
>
> Ordering by type from largest to smallest can save some gas for struts
> in storage, such as the Auction struct found in shortfall.sol
>
> Here is the suggested ordering; it’s good practice to pack uint256 and
> address types first as they take up full 32 byte storage slots.

```
struct Auction {
    uint256 startBlock;
    uint256 seizedRiskFund;
    uint256 highestBidBps;
    uint256 highestBidBlock;
    uint256 startBidBps;
    address highestBidder;
    AuctionType auctionType;
    AuctionStatus status;
    VToken[] markets;
    mapping(VToken => uint256) marketDebt;
}
```

## Redundant call

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L322](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L322)

```
~~token.safeApprove(address(vToken), 0);~~
token.safeApprove(address(vToken), amountToSupply);
```

The idea of calling this twice is to minimise risk of front-running -
assuming the allowance is non-zero to begin with, an actor could spend
all of the allowance before the transaction is mined, then essentially
have the allowance replenished by the safeApprove call.

This is not necessary in this specific case, for two reasons:

-   These are called within the same transaction; if the allowance was
    > previously non-zero, then it could still be front-run

-   The vToken being approved has been just created, also in the same
    > transaction. An existing approval for it will not exist

## Using calldata instead of memory for array function params can save gas:

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154)

[https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L197](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L197)
