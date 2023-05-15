# Code4rena Venus Protocol May 2023 - Team_Rocket QA Report

## Summary

The codebase is well coded, but could improve in adding sanity checks to make sure that user/admin inputs are intended. The codebase can also provide more informational comments about the purpose of some state variables to improve readability.

## Low Severity Findings

### [L-01] Add a sanity check in `addMarket` to make sure that the correct interest rate model is being used

#### Location

[PoolRegistry.sol#L256-L281](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L256-L281)

#### Description

The `addMarket` function allows the `Comptroller` contract to add a new market that follows either the Jump Rate or the Whitepaper interest rate model. These two models have different parameters that are given to the function through the `AddMarketInput` struct.

The wrong interest rate model could be used to create a new market due to incorrect input. For example, a market was intended to use the Jump Rate model but through incorrect input, is set to use the Whitepaper model.

#### Impact

The wrong interest rate model could be used for a market.

#### Tools Used

Manual review

#### Recommended Mitigation Steps

Include a check for `kink` and `jumpMultiplier` to be equal to zero if the market is intended to be based on the Whitepaper model.

```diff
@@ -277,6 +277,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
                 )
             );
         } else {
+            require(input.kink_ == 0 && input.jumpMultiplierPerYear == 0, "PoolRegistry: Invalid Whitepaper parameters");
             rate = InterestRateModel(whitePaperFactory.deploy(input.baseRatePerYear, input.multiplierPerYear));
         }
```

### [L-02] `RewardsDistributor._grantRewardToken` return values are confusing

#### Location

[RewardsDistributor.sol#L416-L423](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L416-L423)

#### Description

The `_grantRewardToken` function returns a `uint256` value that is either `0` for a successful transfer of reward tokens or `amount` for an unsuccessful transfer. This is confusing, and can lead to vulnerabilities in the future if the contract was upgraded.

#### Tools Used

Manual review

#### Recommended Mitigation Steps

Use a `bool` value indicating whether the transfer of reward tokens for successful. Refactor `claimRewardToken(address,VToken[])` and `grantRewardToken` to account for the new `bool` return value.

## Non-Critical Findings

### [NC-01] Incorrect comment to describe `protocolShareReserve` state variable

#### Location

[PoolRegistry.sol#L75-L77](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L75-L77)

#### Description

The comment that describes the `protocolShareReserve` state variable is incorrect and actually describes the `shortfall` state variable.

#### Tools Used

Manual review

#### Recommended Mitigation Steps

Fix the comment to correctly describe `protocolShareReserve`.

```diff
@@ -73,7 +73,7 @@ contract PoolRegistry is Ownable2StepUpgradeable, AccessControlledV8, PoolRegist
     Shortfall public shortfall;

     /**
-     * @notice Shortfall contract address
+     * @notice ProtocolShareReserve contract address
      */
     address payable public protocolShareReserve;
```
