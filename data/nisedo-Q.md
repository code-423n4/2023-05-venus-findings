# Named imports can be used

#### **Context:** All contracts

#### **Description:**
It’s possible to name the imports to improve code readability.

E.g. `import “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol”;`

can be rewritten as:

`import {Ownable2StepUpgradeable} from “@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol;”`

E.g. **FILE: contracts/Comptroller.sol**

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

#### **Recommendation:**
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section. (https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length)

# Missing NatSpec & NatSpec comments should be increased in contracts

#### **Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. (https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)


**FILE:2023-05-venus/contracts/ComptrollerInterface.sol**
[ComptrollerInterface.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerInterface.sol)

**FILE: 2023-05-venus/contracts/Factories/VTokenProxyFactory.sol**
[VTokenProxyFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Factories/VTokenProxyFactory.sol)

**FILE: 2023-05-venus/contracts/ErrorReporter.sol**
[ErrorReporter.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ErrorReporter.sol)

**FILE: 2023-05-venus/contracts/Factories/JumpRateModelFactory.sol**
[JumpRateModelFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Factories/JumpRateModelFactory.sol)

**FILE: 2023-05-venus/contracts/RiskFund/IRiskFund.sol**
[IRiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/IRiskFund.sol)

**FILE: 2023-05-venus/contracts/IPancakeswapV2Router.sol**
[IPancakeswapV2Router.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/IPancakeswapV2Router.sol)

**FILE: 2023-05-venus/contracts/Factories/WhitePaperInterestRateModelFactory.sol**
[WhitePaperInterestRateModelFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Factories/WhitePaperInterestRateModelFactory.sol#L6-L12)

**FILE: 2023-05-venus/contracts/Proxy/UpgradeableBeacon.sol**
[UpgradeableBeacon.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Proxy/UpgradeableBeacon.sol)

**FILE: 2023-05-venus/contracts/RiskFund/IProtocolShareReserve.sol**
[IProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/IProtocolShareReserve.sol)

**FILE: 2023-05-venus/contracts/Shortfall/IShortfall.sol**
[IShortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/IShortfall.sol)