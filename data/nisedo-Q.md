# Named imports can be used

**Context:** All contracts

**Description:**
It’s possible to name the imports to improve code readability.

E.g. `import “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol”;`

can be rewritten as:

`import {Ownable2StepUpgradeable} from “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol;”`

E.g. FILE: contracts/Comptroller.sol

[Comptroller.sol#L4-L12](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L4-L12)

# Long lines are not suitable for the ‘Solidity Style Guide’

**FILE: 2023-05-venus/contracts/Comptroller.sol**

- https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L419

- https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L827

- https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L853

**FILE: 2023-05-venus/contracts/VToken.sol**
- https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L797-L798

**FILE: 2023-05-venus/contracts/Shortfall/Shortfall.sol**
- https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L329

**Recommendation:**
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section. (https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length)