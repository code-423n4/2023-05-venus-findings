# Named imports can be used

**Context:** All contracts

**Description:**
It’s possible to name the imports to improve code readability.

E.g. `import “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol”;`

can be rewritten as:

`import {Ownable2StepUpgradeable} from “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol;”`

E.g. FILE: contracts/Comptroller.sol

[Comptroller.sol#L4-L12](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L4-L12)