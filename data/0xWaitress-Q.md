1. 2 decimal/precision in baseUnit in ProtocolShareReserve is too little.

```solidity
    // Percentage of funds not sent to the RiskFund contract when the funds are released, following the project Tokenomics
    uint256 private constant protocolSharePercentage = 70;
    uint256 private constant baseUnit = 100;
```
Since the unit is used to calculate the ratio to send to protocolIncome / riskFund, right now with 2 decimals if the protocolIncome would have at worst, a floor down of 1%. It is advised to raise the decimals to at least 4-5 so the protocolIncome would not get impacted by division math. 

```solidity
        uint256 protocolIncomeAmount = mul_(
            Exp({ mantissa: amount }),
            div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
        ).mantissa;
```


2. the placebid function in Shortfall is unnecessarily gas intensive since some tokens with auction.marketDebt[auction.markets[i] == 0 can be skipped from transferFrom. Since not necessarily all markets have bad debt in an auction.

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