[L-1] Incorrect error message

Instances(1):

```
    require(
        (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
            ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
            (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
        "your bid is not the highest"
    );
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L171

Mitigation:

```
    auction.auctionType == AuctionType.LARGE_POOL_DEBT ? "your bid is not the highest" : "your bid is not the lowest"
```

[N-1] Spellcheck

Instances (5):

market's -> market

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L300

a -> an

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L88
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L356
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#LL99

used -> used to

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L288

[G-1] Unnecessary transfer

Unnecessary transfer if msg.sender is same as the highest bidder

```
    if (auction.highestBidder != address(0)) {
        erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
    }

    erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L189-L193

Check if the current highest bidder is placing a new bid, and skip performing the unnecessary transfers.