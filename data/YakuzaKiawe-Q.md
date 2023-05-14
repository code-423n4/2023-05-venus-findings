# Report
## Low Risk ##
### [L-1]: Negative Utilization Rate
**Context:**

1. https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L96

**Description:**

In `WhitePaperInterestRateModel.sol`, if the reserves are greater than `cash + borrows` , the utilization rate could become negative

**Recommendation:**

Include a check like
`require(reserves < cash + borrows)`

### [L-2]: Same value for 2 variables 
**Context:**

1. https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#LL137C1-L138C1

**Description:**

In `ShortFall.sol` the addresses `convertibleBaseAsset_` and `riskFund_` could be same.
**Recommendation:**

Add a check for this like 
`require(address(riskFund_) !=  convertibleBaseAsset_);`

### [L-3]: transferOwnership should be two step process
**Context:**

1. https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L244

**Description:**

In `ShortFall.sol` auction.highestBidBps could become equal to MAX_BPS if set to 10000.
This could neglect the if-else block 

**Recommendation:**

Implement the if-else block in such a way that they could not become equal to each other.