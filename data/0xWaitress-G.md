1. uncheck ++i;

instead of 
```solidity

        for (uint256 i; i < marketsCount; ++i) {
            ...
        }
```
do
```solidity
        for (uint256 i; i < marketsCount;) {
            ...
           unchecked {
               ++i
        }
```

There are 28 places across ShortFall, RewardDistributor, RiskFund, PoolRegistry and Comptroller that can be improved.

--- \n


2. the placebid function in Shortfall is unnecessarily gas intensive since some tokens with `auction.marketDebt[auction.markets[i] == 0` can be skipped from the function `transferFrom`. Since not necessarily all markets have bad debt in an auction.

```solidity
        for (uint256 i; i < marketsCount; ++i) {
            VToken vToken = VToken(address(auction.markets[i]));
            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
                if (auction.highestBidder != address(0)) {
                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
                        MAX_BPS);
                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
                }

                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
```

Recommendation

```solidity
+++ if (auction.marketDebt[auction.markets[i]] >0) ....
```