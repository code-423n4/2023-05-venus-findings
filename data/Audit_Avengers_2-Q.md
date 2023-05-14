https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L998-L1002

consider the code block below from the Vtoken internal function  _liquidateBorrow.

        function _liquidateBorrow(
        address liquidator,
        address borrower,
        uint256 repayAmount,
        VTokenInterface vTokenCollateral,
        bool skipLiquidityCheck
    ) internal nonReentrant {
        accrueInterest();

        uint256 error = vTokenCollateral.accrueInterest();
        if (error != NO_ERROR) {  //@audit whats the point of this if block? accruss interest only returns NO_ERROR
            // accrueInterest emits logs on errors, but we still want to log the fact that an attempted liquidation failed
            revert LiquidateAccrueCollateralInterestFailed(error);
        }

what is the purpose of the if block?  Looks like either its not intended to return nothing else but NO_ERROR, and this was overlooked by the devs, or that the if block is redundant, in which case it will waste gas.