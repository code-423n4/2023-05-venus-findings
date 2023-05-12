##CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS
Make it outside and only use it inside.
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L162-L168
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L611-L626
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L389-L399
```
        for (uint256 i; i < userAssetsCount; ++i) {
            VToken market = userAssets[i];

            (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);
            uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);

            // Seize the entire collateral
            if (tokens != 0) {
                market.seize(liquidator, user, tokens);
            }
            // Repay a certain percentage of the borrow, forgive the rest
            if (borrowBalance != 0) {
                market.healBorrow(liquidator, user, repaymentAmount);
            }
        }
    }
```