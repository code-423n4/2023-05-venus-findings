| ID     |                                         Title                                          | Instances |
| :----- | :------------------------------------------------------------------------------------: | --------: |
| [L-01] |           Check if `baseRatePerYear > blocksPerYear` to avoid division-error           |         1 |
| [L-02] | Use `require()` statement to prevent irregular flow of `marketCount` and `actionCount` |         1 |
| [L-03] |           Unecessary use of extra variables results in slow process of code            |         1 |
| [L-04] |             Comment for `protocolShareReserve` variable should be changed              |         1 |

### [L-01] Check if `baseRatePerYear > blocksPerYear` to avoid division-error

Here we need to check if `baseRatePerYear > blocksPerYear` to avoid division-error.

```solidity
BaseJumpRateModelV2.sol

 function _updateJumpRateModel(
        uint256 baseRatePerYear,
        uint256 multiplierPerYear,
        uint256 jumpMultiplierPerYear,
        uint256 kink_
    ) internal {
        baseRatePerBlock = baseRatePerYear / blocksPerYear; // @audit check if baseRatePerYear > blocksPerYear to avoid division-error
        multiplierPerBlock = (multiplierPerYear * BASE) / (blocksPerYear * kink_);
        jumpMultiplierPerBlock = jumpMultiplierPerYear / blocksPerYear;
        kink = kink_;

        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);
    }
```

### [L-02] Use `require()` statement to prevent irregular flow of `marketCount` and `actionCount`

what if marketIdx > actionIdx or any combination that cause irregular flow @discuss marketCount and actionsCount using `require`.

```solidity
    function setActionsPaused(VToken[] calldata marketsList, Action[] calldata actionsList, bool paused) external {
        _checkAccessAllowed("setActionsPaused(address[],uint256[],bool)");

        uint256 marketsCount = marketsList.length;
        uint256 actionsCount = actionsList.length;

        _ensureMaxLoops(marketsCount);

        for (uint256 marketIdx; marketIdx < marketsCount; ++marketIdx) {
            // @audit what if marketIdx > actionIdx or any combination that cause irregular flow @discuss or check marketCount and actionsCount using `require`
            for (uint256 actionIdx; actionIdx < actionsCount; ++actionIdx) {
                _setActionPaused(address(marketsList[marketIdx]), actionsList[actionIdx], paused);
            }
        }
    }
```

Also here `require()` statement is necessary to avoid `underflow`.

```solidity
function releaseFunds(address comptroller, address asset, uint256 amount) external returns (uint256) {
        require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
        require(amount <= poolsAssetsReserves[comptroller][asset], "ProtocolShareReserve: Insufficient pool balance");

        assetsReserves[asset] -= amount; // @note
        poolsAssetsReserves[comptroller][asset] -= amount; // @note
        uint256 protocolIncomeAmount = mul_(
            Exp({ mantissa: amount }),
            div_(Exp({ mantissa: protocolSharePercentage * expScale }), baseUnit)
        ).mantissa;

        IERC20Upgradeable(asset).safeTransfer(protocolIncome, protocolIncomeAmount);
        IERC20Upgradeable(asset).safeTransfer(riskFund, amount - protocolIncomeAmount); // @audit this will underflow if the amount is less than protocolIncomeAmount -> this will drain the funds to riskFund may be

        // Update the pool asset's state in the risk fund for the above transfer.
        IRiskFund(riskFund).updateAssetsState(comptroller, asset); // @audit before updation of state the attacker may call a withdraw function or will call a function that contrain asset for his own benefit

        emit FundsReleased(comptroller, asset, amount);

        return amount;
    }
```

### [L-03] Unecessary use of extra variables results in slow process of code

Comptroller.sol

```solidity
   uint256 rewardsDistributorsLength = rewardsDistributors.length;// 1st time

        for (uint256 i; i < rewardsDistributorsLength; ++i) {
            address rewardToken = address(rewardsDistributors[i].rewardToken());
            require(
                rewardToken != address(_rewardsDistributor.rewardToken()),
                "distributor already exists with this reward"
            );
        }

        uint256 rewardsDistributorsLen = rewardsDistributors.length;// again here
```

### [L-04] Comment for `protocolShareReserve` variable should be changed

Here `comment must be //protocolShareReserve address`

```solidity
 /**
     * @notice Shortfall contract address
     */
    Shortfall public shortfall;

    /**
     * @notice Shortfall contract address // @audit this comment should be changed
     */
    address payable public protocolShareReserve;

```
