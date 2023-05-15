## Summary

### Low Risk Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| L-01 | MaxLoopsLimitHelper is inefficient | 1   |
| L-02 | decimals() function is not guaranteed in ERC20 | 1   |
| L-03 | Miscalculated gap size | 3 |
| L-04 | Anyone could claim rewards on others' behalf | 1 |
| L-05 | The owner could bypass withdraw check in case of multi-address token | 1 |

### Non-Critical Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| N-01 | Wrong/misleading comments | 3  |
| N-02 | Redundant check | 1  |

* * *

### L-01  MaxLoopsLimitHelper is inefficient

Multiple times in protocol there are calls to `MaxLoopsLimitHelper#_ensureMaxLoops` function that is designed to avoid DOS fails:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L39
```solidity
File: MaxLoopsLimitHelper.sol
34:     /**
35:      * @notice Comapre the maxLoopsLimit with number of the times loop iterate
36:      * @param len Length of the loops iterate
37:      * @custom:error MaxLoopsLimitExceeded error is thrown when loops length exceeds maxLoopsLimit
38:      */
39:     function _ensureMaxLoops(uint256 len) internal view {
40:         if (len > maxLoopsLimit) {
41:             revert MaxLoopsLimitExceeded(maxLoopsLimit, len);
42:         }
43:     }
```

However, instead of preventing such failures, this functionality would only increase cases when DOS occurs. For example, in the function `RewardsDistributor#claimRewardToken()`:

```solidity
File: RewardsDistributor.sol
277:     function claimRewardToken(address holder, VToken[] memory vTokens) public {
278:         uint256 vTokensCount = vTokens.length;
279: 
280:         _ensureMaxLoops(vTokensCount);
281: 
282:         for (uint256 i; i < vTokensCount; ++i) {
283:             VToken vToken = vTokens[i];
284:             require(comptroller.isMarketListed(vToken), "market must be listed");
285:             Exp memory borrowIndex = Exp({ mantissa: vToken.borrowIndex() });
286:             _updateRewardTokenBorrowIndex(address(vToken), borrowIndex);
287:             _distributeBorrowerRewardToken(address(vToken), holder, borrowIndex);
288:             _updateRewardTokenSupplyIndex(address(vToken));
289:             _distributeSupplierRewardToken(address(vToken), holder);
290:         }
291:         rewardTokenAccrued[holder] = _grantRewardToken(holder, rewardTokenAccrued[holder]);
292:     }
```

If DOS would occur with `vTokensCount` greater than 50 and current `maxLoopsLimit` is 40, then DOS would take place inside MaxLoopsLimitHelper#_ensureMaxLoops on any `vTokensCount` on any value > 40. While it would be fine up to 50 if MaxLoopsLimitHelper wasn't used. 

Recommendation: Consider removing MaxLoopsLimitHelper functionality in places where it only provokes DOS.

### L-02 decimals() function is not guaranteed in ERC20
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L284
```solidity
File: PoolRegistry.sol
284:         uint256 underlyingDecimals = IERC20Metadata(input.asset).decimals();
```
According to the [EIP-20](https://eips.ethereum.org/EIPS/eip-20), the `decimals()` function is considered optional for ERC-20 tokens, meaning that tokens without this function may be unable to participate in certain protocol functionalities. Recommendation: Consider updating function that calls `decimals()` function.

### L-03 Miscalculated gap size 
The conventional size for the "gap" of unused storage slots, reserved for new variables is 50 minus the number of slots already used. In multiple places in scope this convention is broken:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L26
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L122
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L130
Recommendation: 
Consider updating the "gap" size to appropriate numbers.

### L-04 Anyone could claim rewards on others' behalf

`RewardsDistributor#claimRewardToken()` allows claim rewards tokens with arbitrary holder address:
```solidity
File: RewardsDistributor.sol
237:     /**
238:      * @notice Claim all the rewardToken accrued by holder in all markets.
239:      * @param holder The address to claim REWARD TOKEN for
240:      */
241:     function claimRewardToken(address holder) external { 
242:         return claimRewardToken(holder, comptroller.getAllMarkets());
243:     }
```
While it's not lead to direct loss of funds, it could be used for griefing attacks in terms of taxes for example by forcing claim rewards when user don't interested in that.
Recommendation: Consider allowing reward claiming only for own `msg.sender`.

### L-05 The owner could bypass withdraw check in case of multi-address token

Token#sweepToken allows the contract owner to withdraw  ERC-20 tokens accidentally sent to the contract address. This function includes check that the withdrawing token is not an underlying, preventing potential rug-pull:

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L526
```solidity
File: VToken.sol
519:     /**
520:      * @notice A public function to sweep accidental ERC-20 transfers to this contract. Tokens are sent to admin (timelock)
521:      * @param token The address of the ERC-20 token to sweep
522:      * @custom:access Only Governance
523:      */
524:     function sweepToken(IERC20Upgradeable token) external override { 
525:         require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");
526:         require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");
527:         uint256 balance = token.balanceOf(address(this));
528:         token.safeTransfer(owner(), balance);
529: 
530:         emit SweepToken(address(token));
531:     }
```

However, this check would be inefficient for multi-address tokens, since it checks only one address and the owner could use the other one for a call and withdraw all underlying balances.

Recommendation: Consider comparing the balance of underlying tokens before and after the sweep, this would prevent withdrawing underlying tokens with multiple addresses.

### N-01 Wrong/misleading comments

Comment refers to other varialbe:
```solidity
File: PoolRegistry.sol
75:     /**
76:      * @notice Shortfall contract address 
77:      */
78:     address payable public protocolShareReserve;

```

Comments inform about possible transfers, while functions don't call `transfer()`:
```solidity
File: RewardsDistributor.sol
335:     /**
336:      * @notice Calculate REWARD TOKEN accrued by a supplier and possibly transfer it to them. 
337:      * @param vToken The market in which the supplier is interacting
338:      * @param supplier The address of the supplier to distribute REWARD TOKEN to
339:      */
340:     function _distributeSupplierRewardToken(address vToken, address supplier) internal {
```
```solidity
File: RewardsDistributor.sol
369:     /**
370:      * @notice Calculate reward token accrued by a borrower and possibly transfer it to them.
371:      * @param vToken The market in which the borrower is interacting
372:      * @param borrower The address of the borrower to distribute REWARD TOKEN to
373:      */
374:     function _distributeBorrowerRewardToken(
```

### N-02 Redundant check 

No needed zero check here, since address would be checked during OZ `UpgradeableBeacon` constructor call: 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol#L8
```solidity
File: UpgradeableBeacon.sol
7:     constructor(address implementation_) UpgradeableBeacon(implementation_) {
8:         require(implementation_ != address(0), "Invalid implementation address"); 
9:     }
```