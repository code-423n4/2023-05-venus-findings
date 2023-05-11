L-01:`setShortfallContract()` May cause old shortfall to be in bidding and cannot be closed forever

The current protocol VToken can set a new `shortfall` with `setShortfallContract()`.
So that if the old `shortfall` has a bid in progress
`vToken.badDebtRecovered()` on vToken will fail, because there is no permission and `shortfall` has been changed
The auction cannot be closed, `closeAuction()` will revert
The user's money may be lost
Suggest adding a mechanism to determine that the old shortfall does not have the `vToken` in the auction




L-02:preLiquidateHook() Missing trigger rewardsDistributors for `borrower` index update

`Liquidate` It is recommended to update the index of `borrower` even if it is liquidated.

```solidity
    function preLiquidateHook(
        address vTokenBorrowed,
        address vTokenCollateral,
        address borrower,
        uint256 repayAmount,
        bool skipLiquidityCheck
    ) external override {
....


+        uint256 rewardDistributorsCount = rewardsDistributors.length;
+        for (uint256 i; i < rewardDistributorsCount; ++i) {
+            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
+            rewardsDistributors[i].updateRewardTokenBorrowIndex(vTokenBorrowed, borrowIndex);
+            rewardsDistributors[i].distributeBorrowerRewardToken(vTokenBorrowed, borrower, borrowIndex);
+        }        
    }    
```

L-03:transfer() missing `accrueInterest()` first

When the user performs `transfer()`, it will check if there is enough collateral.
However, the current transfer is not executed first `accrueInterest()`, so the amount owed by the user may be less than the actual amount owed, resulting in a transfer that would otherwise be impossible.

It is recommended to execute `accrueInterest()` first when transferring

```solidity
    function transfer(address dst, uint256 amount) external override nonReentrant returns (bool) {
+       accrueInterest();
        _transferTokens(msg.sender, msg.sender, dst, amount);
        return true;
    }

    function transferFrom(
        address src,
        address dst,
        uint256 amount
    ) external override nonReentrant returns (bool) {
+       accrueInterest();    
        _transferTokens(msg.sender, src, dst, amount);
        return true;
    }
```    

L-04: healBorrow() lacks `accrueInterest()` first

healBorrow() will process the amount owed by the user first, currently there is no `accrueInterest()`, the actual operation of the amount owed may become less

```solidity
    function healBorrow(
        address payer,
        address borrower,
        uint256 repayAmount
    ) external override nonReentrant {
        if (msg.sender != address(comptroller)) {
            revert HealBorrowUnauthorized();
        }
+       accrueInterest();
```


L-05:sweepToken() The underlying address is invalid for a token with a double address

`sweepToken()` will restrict the transfer of `underlying`.
But currently it is a comparison address, this restriction will be invalid for some tokens with proxy address, we suggest to check wether the balance of `underlying` has changed.

https://github.com/d-xo/weird-erc20/#multiple-token-addresses

```solidity
    function sweepToken(IERC20Upgradeable token) external override {
        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
        require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
        uint256 balance = token.balanceOf(address(this));
+       uint256 underlyingBalance = underlying.balanceOf(address(this)); 
        token.safeTransfer(owner(), balance);
+       require(underlying.balanceOf(address(this)) == underlyingBalance,"can not sweep underlying token");
        emit SweepToken(address(token));
    }
```

L-06: The redeemTokens calculation for _redeemFresh() is recommended to use `round up`

According to the ERC4626 standard, for protocol security, the tokens fetched by the asset calculation should be `round up ` when redeeming

```solidity
    function _redeemFresh(
        address redeemer,
        uint256 redeemTokensIn,
        uint256 redeemAmountIn
    ) internal {
...

            redeemTokens = div_(redeemAmountIn, exchangeRate);
+           if(redeemTokens * exchangeRate !=  redeemAmountIn) redeemTokens++;// round up            
            redeemAmount = redeemAmountIn;
        }    
```


L-07:updateNextBidderBlockLimit() will break the end mechanism already in the auction

`updateNextBidderBlockLimit()` will modify the protocol `nextBidderBlockLimit`.
This value is very important for bidding, to indicate whether the current bidder has already bid successfully
However, the current protocol does not record the `nextBidderBlockLimit` for bids that have already started.
This leads to the fact that if the old auction has already started, the bidder will think that the calculation time is: `nextBidderBlockLimit==100000`.

But then the protocol is changed to `nextBidderBlockLimit==10`
So the old bidder also becomes 10, not at all the 100000 that the bidder saw at that time
This corresponds to the user is not fair
So it is recommended that each bid add a record of the `nextBidderBlockLimit` time at the time, the new modification, and will not affect the old

```solidity
    struct Auction {
        uint256 startBlock;
        AuctionType auctionType;
        AuctionStatus status;
        VToken[] markets;
        uint256 seizedRiskFund;
        address highestBidder;
        uint256 highestBidBps;
        uint256 highestBidBlock;
        uint256 startBidBps;
        mapping(VToken => uint256) marketDebt;
+       uint256 nextBidderBlockLimit;
    }

```
