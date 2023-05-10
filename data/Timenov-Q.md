# Venus report by Timenov

## Summary
L-01 Wrong `baseRatePerBlock` and `multiplierPerBlock` during leap years.

### [L-01] Wrong `baseRatePerBlock` and `multiplierPerBlock` during leap years.
*There are 2 instances of this issue.*

The `baseRatePerBlock` and `multiplierPerBlock` depend on the `blocksPerYear` which is a `constant`, so the constant will have the wrong value during leap years.

```solidity
File: contracts/WhitePaperInterestRateModel.sol

17: uint256 public constant blocksPerYear = 2102400;
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L17

```solidity
File: contracts/BaseJumpRateModelV2.sol

23: uint256 public constant blocksPerYear = 10512000;
```

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#LL23C5-L23C54