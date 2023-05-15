## [G‑01] Move the validation if `snapshot.shortfall` is equals to 0 right after the snapshot has been calculated
- These check can be done immediatelly after the snapshot has been calculated because if the shortfall is equals to 0, the liquidation process will end up being aborted, so, there is no a real benefit on doing any calculations before determing if the position/account can be liquidated.
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L464
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L595
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L661
### Recommendation
- The recommendation it is to move the validation of the shortfall right after the snapshot has been calculated, all the times that a liquidation must be reverted because the position/account is safe, the users will save all the gas from the calculations that were made before validation if the shortfall was equals to 0


## [G‑02] Check if marketBadDebt is != 0, if not, no need to write to storage
- When starting an auction in the `Shortfall:_startAuction()` function, at the moment of registering the debts of the individual markets it is [written to storage when assigning the `marketBadDebt` to `auction.marketDebt[vTokens[i]]`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L397), the thing is that **not all the markets of a pool will have a debt**, thus, if the value of `marketBadDebt` is 0, that action of writting to storage could be skipped, in the end, the default value of an uint is 0.
### Recommendation
- The recommendation it is to check if `marketBadDebt is != 0`, if not, **there is no need to write to storage a 0**, it can be left unnassigned it, default uint value is 0