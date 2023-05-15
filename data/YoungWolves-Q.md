# [L-01] `baseRatePerYear` needs to be calculated differently for Jump- and WhitePaperInterest- Rate Models
    
#### Files: 
**PoolRegistry.sol**
[L272](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L272)
[L280](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L280)
    
**BaseJumpRateModelV2.sol**
[L23](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#LL23C54-L23C54)
    
**WhitePaperInterestRateModel.sol**
[L17](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#LL17C45-L17C52)
    
#### Description:
    
`blocksPerYear` is used as a constant in **BaseJumpRateModelV2.sol** and **WhitePaperInterestRateModel.sol**
    
While Venus has been forked from compound, they are deloyed on different networks BSC an Ethereum respectively. The difference in average block time is about 4x (3s for BSC, 12s for Ethereum).
    
In **BaseJumpRateModelV2.sol** the value of `blocksPerYear` has been adjusted for the network change to be equal to `10512000`, in **WhitePaperInterestRateModel.sol** the value remains the same as in compound's original: `2102400`
    
To confirm that this is indeed a mistake, here is a [link to the current Venus github](https://github.com/VenusProtocol/venus-protocol/blob/f1426f07b44f9b557059a0e364ce5d90e18f997a/contracts/InterestRateModels/WhitePaperInterestRateModel.sol#L19), where this value is calculated based on block time.
    
Taking it into consideration, when calling PoolRegistry#addMarket() the `baseRatePerYear` would have to be calculated differently, taking into account different values as mentioned above, to achieve the same rates.

# [L-02] Difference between `blocksPerYear` hardcoded contant value in Jump- and WhitePaperInterest- Rate Models could lead to DOS if not spotted
    
#### Files:
**PoolRegistry.sol**
[L272](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L272)
[L280](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L280)

**BaseJumpRateModelV2.sol**
[L23](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#LL23C54-L23C54)

**WhitePaperInterestRateModel.sol**
[L17](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#LL17C45-L17C52)
[L56](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L56) 

**VToken.sol**
L [158](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L158), [168](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L168), [181](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L181), [198](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L198), [214](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L214), [227](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L227), [242](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L242), [256](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L256), [271](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L271), [333](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L333), [346](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L346), [357](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L357), [372](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L372), [666](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L666),
[L695-696](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L695-L696)

#### Description:

`blocksPerYear` is used as a constant in **BaseJumpRateModelV2.sol** and **WhitePaperInterestRateModel.sol**

While Venus has been forked from compound, they are deloyed on different networks BSC an Ethereum respectively. The difference in average block time is about 4x (3s for BSC, 12s for Ethereum).

In **BaseJumpRateModelV2.sol** the value of `blocksPerYear` has been adjusted for the network change to be equal to `10512000`, in **WhitePaperInterestRateModel.sol** the value remains the same as in compound's original: `2102400`

To confirm that this is indeed a mistake, here is a [link to the current Venus github](https://github.com/VenusProtocol/venus-protocol/blob/f1426f07b44f9b557059a0e364ce5d90e18f997a/contracts/InterestRateModels/WhitePaperInterestRateModel.sol#L19), where this value is calculated based on block time:

If unnoticed, it could cause a market-wide DOS 5 times faster than expected:

In VToken.sol 14 different functions include `accrueInterest()` , which in turn checks the result of `InterestRateModel.getBorrowRate()` against the `borrowRateMaxMantissa` (L695-696). In **WhitePaperInterestRateModel.sol** `getBorrowRate()` returns `((ur * multiplierPerBlock) / BASE) + baseRatePerBlock` where both `multiplierPerBlock` and `baseRatePerBlock` are dependant on `blocksPerYear`.


# [NC-01] Extract duplicated code to a function

#### File:
**ReserveHelpers.sol**
[L39-40](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L39-L40)
[L50-51](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L50-L51)

Such as:
```
modifier checkIsValidAddress private {
  require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");
  require(asset != address(0), "ReserveHelpers: Asset address invalid");
  _;
}
```

# [NC-02] Redundant Openzeppelin's Ownable2StepUpgradeable.sol import

#### Files:
**VToken.sol** [L4](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L4)
**PoolRegistry.sol** [L4](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L4)
**RewardDistributor.sol** [L4](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L4)
**RiskFund.sol** [L4](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L4)
**Shortfall.sol** [L4](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L4)

#### Description:

In all above mentioned contracts the following line:
import `@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol`;
is redundant as all of them include:
import `@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol`;, which itself imports Ownable2StepUpgradeable.sol

# [NC-03] Use inherited function to reduce duplicate code

#### File:
**VToken.sol** [L1363-1364](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1363-L1364)
**PoolRegistry.sol** [L172-173](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L172-L173)
**RewardDistributor.sol** [L119-120](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L119-L120)
**RiskFund.sol** [L85-86](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L85-L86)
**Shortfall.sol** [L141-142](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L141-L142)

**AccessControlledV8.sol** [L34-37](https://github.com/VenusProtocol/governance-contracts/blob/19b431d32fab44cadd49accd429a709e552febce/contracts/Governance/AccessControlledV8.sol#L34-L37)

#### Description:

In above mentioned contract the following code is used during initialization:

```
__Ownable2Step_init();
__AccessControlled_init_unchained(accessControlManager_);
```

It could be in all cases reduced to 1 line, as all the contract inherit from AccessControlledV8, which implements the following function:

```
function __AccessControlled_init(address accessControlManager_) internal onlyInitializing {
    __Ownable2Step_init();
    __AccessControlled_init_unchained(accessControlManager_);
}
```

# [NC-04] Inherited variable shadowed. Consider changing name.

There is already a state variable called `owner` on OwnableUpgradeable contract. It would be convenient to rename that variable's name.

#### File:
**VToken.sol**
[L539](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L539)
[L548](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L548)
