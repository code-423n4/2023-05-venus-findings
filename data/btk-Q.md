# Protocol Overview

| Total Low issues |
|------------------|

| Risk   | Issues Details                                                     | Number        |
|--------|--------------------------------------------------------------------|---------------|
| [L-01] | Lack of Disposal Mechanism in Comptroller.sol                      | 1             |
| [L-02] | A dangerous check in `_setMaxLoopsLimit()`                         | 1             |

## [L-01]  Lack of Disposal Mechanism in Comptroller.sol

#### Impact

The lack of a disposal mechanism in the `Comptroller.sol` contract means that any market that is added to the system cannot be removed later on. This could cause some issues, as markets that are no longer needed or should be removed for other reasons will remain listed forever. This can make the system harder to manage and less efficient, potentially causing some risks in the future.

#### Proof of Concept

[`Comptroller.sol`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol) has a [`supportMarket()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#LL801C1-L824C6) function to add a market to the markets mapping and set it as listed:

```solidity
    function supportMarket(VToken vToken) external {
        _checkSenderIs(poolRegistry);

        if (markets[address(vToken)].isListed) {
            revert MarketAlreadyListed(address(vToken));
        }

        require(vToken.isVToken(), "Comptroller: Invalid vToken"); // Sanity check to make sure its really a VToken

        Market storage newMarket = markets[address(vToken)];
        newMarket.isListed = true;
        newMarket.collateralFactorMantissa = 0;
        newMarket.liquidationThresholdMantissa = 0;

        _addMarket(address(vToken));

        uint256 rewardDistributorsCount = rewardsDistributors.length;

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].initializeMarket(address(vToken));
        }

        emit MarketSupported(vToken);
    }
```

but there is no function to remove a market once its listed.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

We recommend adding a function to remove a market.

## [L-02] A dangerous check in `_setMaxLoopsLimit()`

#### Impact

[`_setMaxLoopsLimit()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32) can't decrease the `maxLoopsLimit` which may impact the protocol if the ethereum gas price go up.

The [`_setMaxLoopsLimit()`](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L25-L32) function is used to set a maximum limit for the number of loops to avoid the Denial-of-Service (DOS) attack. However, the function's check can cause an unintended consequence which may impact the protocol or its availability if the Ethereum gas price goes up.

#### Proof of Concept

`_setMaxLoopsLimit()` function is used to set a max limit for the loops to avoid the DOS:

```solidity
    function _setMaxLoopsLimit(uint256 limit) internal {
        require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

        uint256 oldMaxLoopsLimit = maxLoopsLimit;
        maxLoopsLimit = limit;

        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
    }
```

As you can see, the function checks if the new limit is greater than the current `maxLoopsLimit` value. If it's not, the function will revert. This check prevents reducing the `maxLoopsLimit`, which may not be the intended behavior.

#### Tools Used

Manual Review

#### Recommended Mitigation Steps

Since the `_setMaxLoopsLimit()` function is only callable by trusted entities, we recommend to remove the check from the function which will give the protocol mere flexibility to adjust in te future.
