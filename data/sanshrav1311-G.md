Use require instead of assert to save gas.

Instances include
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL225C9-L225C15


> File: contracts/Comptroller.sol
> 255: assert(assetIndex < len);