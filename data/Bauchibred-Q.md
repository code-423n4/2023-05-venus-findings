
# **Venus QA Report**

## **Table of Contents**

**Low Issues**

|      | Issue                                                                                                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| L-01 | RewardsDistributor.sol: Inconsistency in sister function executionsl                                                                              |
| L-02 | VToken.sol: Wrong steps in the execution of the _redeemFresh() functions                                                                         |
| L-03 | Inconsistent storage gaps                                                                                                                         |
| L-04 | VToken.sol: The error being `NO_ERROR ` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction's execution failed |
| L-05 | RiskFund.sol: Use of block.timestamp for _swapAsset() should be reconsidered                                                                    |
| L-06 | Confusing names of scales in ExponentialNoError.sol                                                                                               |
| L-07 | Missing Zero address checks                                                                                                                       |
| L-08 | Consider isContract checks in constructors                                                                                                        |
| L-09 | Renouncing ownership could break some functionalities                                                                                             |

**Non-Critical Issues**

|       | Issue                                                                                        |
| ----- | -------------------------------------------------------------------------------------------- |
| NC-01 | Comptroller.sol: redundant step in preMintHook()                                             |
| NC-02 | Incomplete event prop                                                                        |
| NC-03 | Typo in docs                                                                                 |
| NC-04 | Some typos were missed by bot                                                                |
| NC-05 | Some events are missing complete indexed fields                                              |
| NC-06 | Comptroller.sol: suggestion to rename minLiquidatableCollateral to maxLiquidatableCollateral |
| NC-08 | uint256 can be used instead of uint                                                          |
| NC-09 | Finally, best practices of source file layout should be followed                             |

# Low Issues

## L-01 RewardsDistributor.sol: Inconsistency in sister function executions

### Vulnerability Details

If you check the code snippet from below you can see that both are sister functions, but in `_distributeBorrowerRewardToken()`, borrowerAmount in this case is been checked to ensure that its not 0 before this calculation goes ahead, where as in the `_distributeSupplierRewardToken()` function no `supplierTokens != 0` checks.

Take a look at the [\_distributeSupplierRewardToken() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L340-L367)

```

function \_distributeSupplierRewardToken((address vToken, address supplier) internal {
RewardToken storage supplyState = rewardTokenSupplyState[vToken];
uint256 supplyIndex = supplyState.index;
uint256 supplierIndex = rewardTokenSupplierIndex[vToken][supplier];

        // Update supplier's index to the current index since we are distributing accrued REWARD TOKEN
        rewardTokenSupplierIndex[vToken][supplier] = supplyIndex;

        if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {
            // Covers the case where users supplied tokens before the market's supply state index was set.
            // Rewards the user with REWARD TOKEN accrued from the start of when supplier rewards were first
            // set for the market.
            supplierIndex = rewardTokenInitialIndex;
        }

        // Calculate change in the cumulative sum of the REWARD TOKEN per vToken accrued
        Double memory deltaIndex = Double({ mantissa: sub_(supplyIndex, supplierIndex) });

        uint256 supplierTokens = VToken(vToken).balanceOf(supplier);

        // Calculate REWARD TOKEN accrued: vTokenAmount * accruedPerVToken
        //@audit no supplierTokens != 0 checks
        uint256 supplierDelta = mul_(supplierTokens, deltaIndex);

        uint256 supplierAccrued = add_(rewardTokenAccrued[supplier], supplierDelta);
        rewardTokenAccrued[supplier] = supplierAccrued;

        emit DistributedSupplierRewardToken(VToken(vToken), supplier, supplierDelta, supplierAccrued, supplyIndex);
    }

```

Now take a look at the [\_distributeBorrowerRewardToken() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L340-L367)

```

    function _distributeBorrowerRewardToken(
        address vToken,
        address borrower,
        Exp memory marketBorrowIndex
    ) internal {
        RewardToken storage borrowState = rewardTokenBorrowState[vToken];
        uint256 borrowIndex = borrowState.index;
        uint256 borrowerIndex = rewardTokenBorrowerIndex[vToken][borrower];

        // Update borrowers's index to the current index since we are distributing accrued REWARD TOKEN
        rewardTokenBorrowerIndex[vToken][borrower] = borrowIndex;

        if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
            // Covers the case where users borrowed tokens before the market's borrow state index was set.
            // Rewards the user with REWARD TOKEN accrued from the start of when borrower rewards were first
            // set for the market.
            borrowerIndex = rewardTokenInitialIndex;
        }

        // Calculate change in the cumulative sum of the REWARD TOKEN per borrowed unit accrued
        Double memory deltaIndex = Double({ mantissa: sub_(borrowIndex, borrowerIndex) });

        uint256 borrowerAmount = div_(VToken(vToken).borrowBalanceStored(borrower), marketBorrowIndex);

        // Calculate REWARD TOKEN accrued: vTokenAmount * accruedPerBorrowedUnit
        //@audit
        if (borrowerAmount != 0) {
            uint256 borrowerDelta = mul_(borrowerAmount, deltaIndex);

            uint256 borrowerAccrued = add_(rewardTokenAccrued[borrower], borrowerDelta);
            rewardTokenAccrued[borrower] = borrowerAccrued;

            emit DistributedBorrowerRewardToken(VToken(vToken), borrower, borrowerDelta, borrowerAccrued, borrowIndex);
        }
    }

```

### Recommendation

The same checks should be used here though i can't really think of an attack window i think execution should be constant for both sister functions

## L-02 VToken.sol: Wrong steps in the execution of the \_redeemFresh() functions

### Vulnerability Details

inconsistency in the execution code block of the \_redeemFresh() function

Take a look at the \_redeemFresh() function:

```

    function _redeemFresh(
        address redeemer,
        uint256 redeemTokensIn,
        uint256 redeemAmountIn
    ) internal {
        require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");

        /* Verify market's block number equals current block number */
        if (accrualBlockNumber != _getBlockNumber()) {
            revert RedeemFreshnessCheck();
        }

        /* exchangeRate = invoke Exchange Rate Stored() */
        Exp memory exchangeRate = Exp({ mantissa: _exchangeRateStored() });

        uint256 redeemTokens;
        uint256 redeemAmount;
        /* If redeemTokensIn > 0: */
        if (redeemTokensIn > 0) {
            /*
             * We calculate the exchange rate and the amount of underlying to be redeemed:
             *  redeemTokens = redeemTokensIn
             *  redeemAmount = redeemTokensIn x exchangeRateCurrent
             */
            redeemTokens = redeemTokensIn;
            redeemAmount = mul_ScalarTruncate(exchangeRate, redeemTokensIn);
        } else {
            /*
             * We get the current exchange rate and calculate the amount to be redeemed:
             *  redeemTokens = redeemAmountIn / exchangeRate
             *  redeemAmount = redeemAmountIn
             */
            redeemTokens = div_(redeemAmountIn, exchangeRate);
            redeemAmount = redeemAmountIn;
        }

        // Require tokens is zero or amount is also zero
        if (redeemTokens == 0 && redeemAmount > 0) {
            revert("redeemTokens zero");
        }

        /* Fail if redeem not allowed */
        comptroller.preRedeemHook(address(this), redeemer, redeemTokens);

        /* Fail gracefully if protocol has insufficient cash */
        if (_getCashPrior() - totalReserves < redeemAmount) {
            revert RedeemTransferOutNotPossible();
        }
                /////////////////////////
        // EFFECTS & INTERACTIONS
        // (No safe failures beyond this point)

        /*
         * We write the previously calculated values into storage.
         *  Note: Avoid token reentrancy attacks by writing reduced supply before external transfer.
         */
        totalSupply = totalSupply - redeemTokens;
        uint256 balanceAfter = accountTokens[redeemer] - redeemTokens;
        accountTokens[redeemer] = balanceAfter;

        /*
         * We invoke _doTransferOut for the redeemer and the redeemAmount.
         *  On success, the vToken has redeemAmount less of cash.
         *  _doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
         */
        _doTransferOut(redeemer, redeemAmount);

        /* We emit a Transfer event, and a Redeem event */
        emit Transfer(redeemer, address(this), redeemTokens);
        emit Redeem(redeemer, redeemAmount, redeemTokens, balanceAfter);
    }

```

So as not to make report unnecessarily bogus take a look at other sister functions, i.e: `\_borrowFresh(), \_repayBorrowFresh(), \_liquidateBorrowFresh(), \_seize(), \_transferTokens(), \_mintFresh() `. In each aforementioned function the first step in the execution of the functions is to query the comptroller to know if the action is allowed, i.e in a case of` _mintfresh()` a preMintHook is checked and would fail if mint is not allowed:

```

comptroller.preMintHook(address(this), minter, mintAmount);

```

And to each other case, their respective hooks are used, now the inconsistency comes in as only in the case of the `_redeemFresh()` is the preRedeemHook not checked as the first step and just doesn't make sense as there is no need to for all other executions if the redeem is not allowed in the comptroller.

Obviously this is submitted as a low as i couldn't find an attack window for this but this just hints a technical flaw.

### Recommendation

In short code should be consistent

## L-03 Inconsistent storage gaps

### Vulnerability Details

When using the proxy pattern for upgrades, it is common practice to include storage gaps on parent contracts to reserve space for potential future variables. The size is typically chosen so that all contracts have the same number of variables (usually 50). However, the code base uses inconsistent sizes and does not always include a gap at all.
for example the `ComptrollerStorage.sol`has gaps set to 50, but the amount gap should be is `50 - "the amount of storage said contract has used"`, same thing happens in `vTokenInterfaces.sol`. I believe it should be 47 in `ReserveHelpers.sol`
[ComptrollerStorage.sol#L122](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L122)
[ReserveHelpers.sol#L26](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L26)

### Recommendation

Consider using consistently sized storage gaps in all contracts that have not yet been deployed so as to facilitate safe upgrades.

## L-04 VToken.sol: The error being `NO_ERROR ` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction failed

### Vulnerability Details

Check out [VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)
In most cases the error is `NO_ERROR ` even in cases it shouldn't be, this means that users can't really know the reasons to why their tx failed, and is a very big flaw for off-chain services

### Recommendation

Though this is done for compatibility with Venus core tooling, this should still undergo a rethink

## L-05 RiskFund.sol: Use of block.timestamp for _swapAsset() should be reconsidered

### Vulnerability Details

There are many instances of block.timestamp usage within the project, most especially in the implementation of the `IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens() `in the `RiskFund._swapAsset()`function block.timestamp can be influenced by miners to a certain degree, so
developers should be aware that this may have some risk if miners collude
on time manipulation to influence the price oracles
Note that this is in relation to the swapForExactTokens function

[RiskFund.\_swapAsset():](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L225-L274)

```

    function _swapAsset(
        VToken vToken,
        address comptroller,
        uint256 amountOutMin,
        address[] calldata path
    ) internal returns (uint256) {
        require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");
        require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");
        uint256 totalAmount;

        address underlyingAsset = VTokenInterface(address(vToken)).underlying();
        uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];

        ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));

        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
            address(vToken)
        );

        if (balanceOfUnderlyingAsset > 0) {
            Exp memory oraclePrice = Exp({ mantissa: underlyingAssetPrice });
            uint256 amountInUsd = mul_ScalarTruncate(oraclePrice, balanceOfUnderlyingAsset);

            if (amountInUsd >= minAmountToConvert) {
                assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
                poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;

                if (underlyingAsset != convertibleBaseAsset) {
                    require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
                    require(
                        path[path.length - 1] == convertibleBaseAsset,
                        "RiskFund: finally path must be convertible base asset"
                    );
                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);
                    uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
                        balanceOfUnderlyingAsset,
                        amountOutMin,
                        path,
                        address(this),
                        //@audit isn't using block.timestamp as a window too short?
                        block.timestamp
                    );
                    totalAmount = amounts[path.length - 1];
                } else {
                    totalAmount = balanceOfUnderlyingAsset;
                }
            }
        }

        return totalAmount;

```

### Recommendation

One could advise that you use `block.number` instead of `block.timestamp` or now to reduce the risk of Maximal Extractable Value (MEV) attacks. Check if the timescale of the
project occurs across years, days and months rather than seconds. If possible, it is recommended to use Oracles.

## L-06 Confusing names of scales in ExponentialNoError.sol

### Vulnerability Details

[ExponentialNoError.sol#L20-L23](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ExponentialNoError.sol#L20-L23)

```

    uint256 internal constant expScale = 1e18;
    uint256 internal constant doubleScale = 1e36;
    uint256 internal constant halfExpScale = expScale / 2;
    uint256 internal constant mantissaOne = expScale;

```

One could argue that the calculation of the scales are somewhat not completely correct, cause as their name suggests

```expScale = 1e18;
doubleScale = 1e36:
halfExpScale = expScale / 2;
```

the third one from the above is not a half scale cause a half scale should be `1e9` but instead here it's just `5e17` which would cause complete lack of correct values where implemented.

Ps: I only submitted this as a low as i couldn't really think of any bug class that could make this exploitable

### Recommendation

Better names should be used or better still comments could be added for more clarification on this

## L-07 Missing Zero address checks

### Vulnerability Details

Multiple instances within contracts in scope, in intializers, constructors, below is the initializer from the PoolRegistry.sol contract:

[Initialize()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L164-L180)

```

function initialize(
VTokenProxyFactory vTokenFactory*,
JumpRateModelFactory jumpRateFactory*,
WhitePaperInterestRateModelFactory whitePaperFactory*,
Shortfall shortfall*,
address payable protocolShareReserve*,
address accessControlManager*
) external initializer {
**Ownable2Step_init();
**AccessControlled*init_unchained(accessControlManager*);

        vTokenFactory = vTokenFactory_;
        jumpRateFactory = jumpRateFactory_;
        whitePaperFactory = whitePaperFactory_;
        _setShortfallContract(shortfall_);
        _setProtocolShareReserve(protocolShareReserve_);
    }

```

### Recommendation

Lack of zero address checks procedure for critical operations leaves them error-prone. Consider adding zero address checks on the critical functions.

## L-08 Consider isContract checks in constructors

### Vulnerability Details

Most of the contracts within scope set addresses based on the value that's been provided to them in the constructor, where as most include checks that the address provided is not 0x0 it does not check that these addresses is a smart contract.
For example take a look at the [UpgradeableBeacon.sol, L6-9](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Proxy/UpgradeableBeacon.sol#L6-L9)

```

    constructor(address implementation_) UpgradeableBeacon(implementation_) {
        require(implementation_ != address(0), "Invalid implementation address");
    }

```

The only check added here is that `address != 0` but if this input gets mistakenly passed with the wrong address, an isContract() check would atleast stop the implementation from being set to the zero address.
Note that this is just one instance and multiple contracts within scope could emulate this as a another layer of protection against setting addresses to the wrong ones

### Recommendation

Consider adding `isContract()` checks

## L-09 Renouncing ownership could break some functionalities

### Vulnerability Details

The Ownable2StepUpgradeable.sol is used and this inherits the Ownable contract which has the `renounceOwnership()` function.

Note that this function breaks the 2 step that's otherwise provided by the Ownable2StepUpgradeable.sol but rather just transfers the ownershp to the zero address in one step which would lead to a break of the whole protocol's logic, most importantly to note that if this gets called, protocol no longer has access to all `onlyOwner() `functions, which means that markets can't get paused incase of an emergency, a new operator can't be set incase access to previous ones gets missing all setters only the owners have acess to can no longer be set etc...

```
    function renounceOwnership() public virtual onlyOwner {
        _transferOwnership(address(0));
    }

```

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/access/Ownable2StepUpgradeable.sol#L20

[Shortfall.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/access/Ownable2StepUpgradeable.sol#L20)

```contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGuardUpgradeable, IShortfall

```

[ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/RiskFund/ProtocolShareReserve.sol#L12)

```
contract ProtocolShareReserve is Ownable2StepUpgradeable, ExponentialNoError, ReserveHelpers, IProtocolShareReserve {

```

And multiple other instances within code.

### Recommendation

Reconsider if the renounceOwnership function is really needed and if not do remove it

##

### Vulnerability Details

### Recommendation

# Non-Critical Issues

## NC-01 Comptroller.sol: redundant step in preMintHook()

### Vulnerability Details

Take a look at the [preMintHook() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L239-L278)

```
    /**
     * @notice Checks if the account should be allowed to mint tokens in the given market
     * @param vToken The market to verify the mint against
     * @param minter The account which would get the minted tokens
     * @param mintAmount The amount of underlying being supplied to the market in exchange for tokens
     * @custom:error ActionPaused error is thrown if supplying to this market is paused
     * @custom:error MarketNotListed error is thrown when the market is not listed
     * @custom:error SupplyCapExceeded error is thrown if the total supply exceeds the cap after minting
     * @custom:access Not restricted
     */
    function preMintHook(
        address vToken,
        address minter,
        uint256 mintAmount
    ) external override {
        _checkActionPauseState(vToken, Action.MINT);

        if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }

        uint256 supplyCap = supplyCaps[vToken];
        // Skipping the cap check for uncapped coins to save some gas
        if (supplyCap != type(uint256).max) {
            uint256 vTokenSupply = VToken(vToken).totalSupply();
            Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
            uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);
            if (nextTotalSupply > supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
        }

        // Keep the flywheel moving
        uint256 rewardDistributorsCount = rewardsDistributors.length;

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
        }
    }
```

As seen at L261, the execution skips the cap check for the uncapped coins to save some gas, I think for a case where the coins are already at their supply caps this execution should also be skipped

### Recommendation

The condition at L261-262 could be changed to:

```
 vTokenSupply =  VToken(vToken).totalSupply() )
if ((supplyCap != type(uint256).max) ||  vTokenSupply != supplyCap){
```

## NC-02 Incomplete event prop

Take a look at \_updateRewardTokenSupplyIndex() of RewardsDistributor.sol:

```
    function _updateRewardTokenSupplyIndex(address vToken) internal {
        RewardToken storage supplyState = rewardTokenSupplyState[vToken];
        uint256 supplySpeed = rewardTokenSupplySpeeds[vToken];
        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
        uint256 deltaBlocks = sub_(uint256(blockNumber), uint256(supplyState.block));
        if (deltaBlocks > 0 && supplySpeed > 0) {
            uint256 supplyTokens = VToken(vToken).totalSupply();
            uint256 accruedSinceUpdate = mul_(deltaBlocks, supplySpeed);
            Double memory ratio = supplyTokens > 0
                ? fraction(accruedSinceUpdate, supplyTokens)
                : Double({ mantissa: 0 });
            supplyState.index = safe224(
                add_(Double({ mantissa: supplyState.index }), ratio).mantissa,
                "new index exceeds 224 bits"
            );
            supplyState.block = blockNumber;
        } else if (deltaBlocks > 0) {
            supplyState.block = blockNumber;
        }
// @audit NC this event should prop up more data like the RewardTokenBorrowIndexUpdated
        emit RewardTokenSupplyIndexUpdated(vToken);
    }
```

And take a look at it's almost opposite function, the \`\_updateRewardTokenBorrowIndex()` function:

```
    function _updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) internal {
        RewardToken storage borrowState = rewardTokenBorrowState[vToken];
        uint256 borrowSpeed = rewardTokenBorrowSpeeds[vToken];
        uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
        uint256 deltaBlocks = sub_(uint256(blockNumber), uint256(borrowState.block));
        if (deltaBlocks > 0 && borrowSpeed > 0) {
            uint256 borrowAmount = div_(VToken(vToken).totalBorrows(), marketBorrowIndex);
            uint256 accruedSinceUpdate = mul_(deltaBlocks, borrowSpeed);
            Double memory ratio = borrowAmount > 0
                ? fraction(accruedSinceUpdate, borrowAmount)
                : Double({ mantissa: 0 });
            borrowState.index = safe224(
                add_(Double({ mantissa: borrowState.index }), ratio).mantissa,
                "new index exceeds 224 bits"
            );
            borrowState.block = blockNumber;
        } else if (deltaBlocks > 0) {
            borrowState.block = blockNumber;
        }

        emit RewardTokenBorrowIndexUpdated(vToken, marketBorrowIndex);
    }
```

As seen the event that gets emitted from the latter has more details being passed out and as such would let off-chain to be able to keep up

### Vulnerability Details

Try including as much data as possible to events

### Recommendation

## NC-03 Typo in docs https://prdt-finance.gitbook.io/prdt-finance-gitbook/smart-contracts/user-accessible-functions

### Vulnerability Details

1. Here is the description to the "claim user function"

```

Allows users to claim their earnings from a specific contract. The user is required to enter the prediction **contract** id and an array of the claimable rounds. All of the rounds entered must be claimable, otherwise the transaction will revert.

```

2. Here under "How to interact with PRDT PRO smart contracts"

```

Step 4: Finish adding a custom ABI by **imputing**:

```

### Recommendation

Change to

1.

```

Allows users to claim their earnings from a specific contract. The user is required to enter the prediction **contract** id and an array of the claimable rounds. All of the rounds entered must be claimable, otherwise the transaction will revert.

```

2.

```

Step 4: Finish adding a custom ABI by **inputing**:

```

## NC-04 Some typos were missed by bot

### Vulnerability Details

1. [RiskFund.sol#L256](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L256)

2. [RewardsDistributor.sol#L411)](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L411)

### Recommendation

1. change error message from

```
"RiskFund: finally path must be convertible base asset"
```

to

```
"RiskFund: FINAL path must be convertible base asset"
```

2. change

```
     * @dev Note: If there is not enough REWARD TOKEN, we do not perform the transfer all.
```

to

```
     * @dev Note: If there is not enough REWARD TOKEN, we do not perform the transfer **AT**all.
```

## NC-05 Some events are missing complete indexed fields

### Vulnerability Details

Multiple instances within code

### Recommendation

Index should be added to events when possible

## NC-06 Comptroller.sol: suggestion to rename minLiquidatableCollateral to maxLiquidatableCollateral

### Vulnerability Details

Take a look at the [preMintHook() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L578-L626)

```
    function healAccount(address user) external {
        VToken[] memory userAssets = accountAssets[user];
        uint256 userAssetsCount = userAssets.length;

        address liquidator = msg.sender;
        // We need all user's markets to be fresh for the computations to be correct
        for (uint256 i; i < userAssetsCount; ++i) {
            userAssets[i].accrueInterest();
            oracle.updatePrice(address(userAssets[i]));
        }

        AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(user, _getLiquidationThreshold);

        if (snapshot.totalCollateral > minLiquidatableCollateral) {
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }

        if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }

        // percentage = collateral / (borrows * liquidation incentive)
        Exp memory collateral = Exp({ mantissa: snapshot.totalCollateral });
        Exp memory scaledBorrows = mul_(
            Exp({ mantissa: snapshot.borrows }),
            Exp({ mantissa: liquidationIncentiveMantissa })
        );

        Exp memory percentage = div_(collateral, scaledBorrows);
        if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
            revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
        }

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

### Recommendation

As seen from function block renaming minLiquidatableCollateral to maxLiquidatableCollateral would massively improve readabilty of code

## NC-07 uint256 can be used instead of uint

### Vulnerability Details

Multiple instances within code in/out of scope, especially in loops

### Recommendation

For explicitness and consistency, uint256 can be used instead uint.

NB: this also eases readability of code

## NC-08 Finally, best practices of source file layout should be followed

### Vulnerability Details

Multiple instances within code in/out of scope

### Recommendation

I recommend following best practices of solidity source file layout for readability.
Reference: https://docs.soliditylang.org/en/v0.8.15/style-guide.html#order-of-layout

This best practices is to layout a contract elements in following order:
Pragma statements, Import statements, Interfaces, Libraries, Contracts

Inside each contract, library or interface, use the following order:
Type declarations, State variables, Events, Modifiers, Functions
