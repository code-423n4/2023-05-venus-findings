### [L-01] Unspecific compiler version pragma

Avoid floating pragmas for non-library contracts.  
It is recommended to pin to a concrete compiler version.  

[contracts/test/ComptrollerHarness.sol#L2](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/ComptrollerHarness.sol#L2)  
```
pragma solidity ^0.8.10;
```
There are 19 instances of this issue.

### [L-02] Do not use deprecated library functions

The usage of deprecated library functions should be discouraged.  
This issue is mostly related to OpenZeppelin libraries.  

[contracts/Pool/PoolRegistry.sol#L322](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Pool/PoolRegistry.sol#L322)  
```
token.safeApprove(address(vToken), 0);
```
[contracts/Pool/PoolRegistry.sol#L323](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Pool/PoolRegistry.sol#L323)  
[contracts/RiskFund/RiskFund.sol#L258](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/RiskFund/RiskFund.sol#L258)  
[contracts/RiskFund/RiskFund.sol#L259](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/RiskFund/RiskFund.sol#L259)  
[contracts/test/Mocks/MockPancakeSwap.sol#L13](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L13)  

### [L-03] `require` should be used instead of `assert`


[contracts/Comptroller.sol#L225](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Comptroller.sol#L225)  
```
assert(assetIndex < len);
```
[contracts/test/Mocks/MockPancakeSwap.sol#L602](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L602)  
[contracts/test/Mocks/MockPancakeSwap.sol#L628](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L628)  
[contracts/test/Mocks/MockPancakeSwap.sol#L692](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L692)  
[contracts/test/Mocks/MockPancakeSwap.sol#L877](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L877)  
[contracts/test/Mocks/MockPancakeSwap.sol#L933](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L933)  
[contracts/test/Mocks/MockPancakeSwap.sol#L995](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/Mocks/MockPancakeSwap.sol#L995)  
[contracts/test/SafeMath.sol#L180](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/test/SafeMath.sol#L180)  

### [L-04] Lines too long

Keep line width to max 120 characters for better readability where possible.  

[contracts/Comptroller.sol#L419](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Comptroller.sol#L419)  
```
* @custom:error MinimalCollateralViolated is thrown if the users' total collateral is lower than the threshold for non-batch liquidations
```
There are 23 instances of this issue.

### [L-05] Not completely using OpenZeppelin contracts

OpenZeppelin maintains a library of standard, audited, community-reviewed, and battle-tested smart contracts. Instead of always importing these contracts, the protocol project re-implements them in some cases. This increases the amount of code that the protocol team will have to maintain and miss all the improvements and bug fixes that the OpenZeppelin team is constantly implementing with the help of the community.  
Consider importing the OpenZeppelin contracts instead of re-implementing or copying them. These contracts can be extended to add the extra functionalities required if need be.  

[contracts/VToken.sol#L33](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/VToken.sol#L33)  
```
modifier nonReentrant() {
```

### [L-06] Use a more recent version of solidity


[contracts/BaseJumpRateModelV2.sol#L2](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/BaseJumpRateModelV2.sol#L2)  
```
pragma solidity 0.8.13;
```
There are 40 instances of this issue.

### [L-07] Constants should be all uppercase with words separated by underscores


[contracts/BaseJumpRateModelV2.sol#L23](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/BaseJumpRateModelV2.sol#L23)  
```
uint256 public constant blocksPerYear = 10512000;
```
[contracts/ComptrollerStorage.sol#L106](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ComptrollerStorage.sol#L106)  
[contracts/ComptrollerStorage.sol#L109](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ComptrollerStorage.sol#L109)  
[contracts/ComptrollerStorage.sol#L112](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ComptrollerStorage.sol#L112)  
[contracts/ComptrollerStorage.sol#L115](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ComptrollerStorage.sol#L115)  
[contracts/ExponentialNoError.sol#L20](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ExponentialNoError.sol#L20)  
[contracts/ExponentialNoError.sol#L21](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ExponentialNoError.sol#L21)  
[contracts/ExponentialNoError.sol#L22](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ExponentialNoError.sol#L22)  
[contracts/ExponentialNoError.sol#L23](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/ExponentialNoError.sol#L23)  
[contracts/InterestRateModel.sol#L10](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/InterestRateModel.sol#L10)  
[contracts/Rewards/RewardsDistributor.sol#L29](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L29)  
[contracts/RiskFund/ProtocolShareReserve.sol#L18](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L18)  
[contracts/RiskFund/ProtocolShareReserve.sol#L19](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L19)  
[contracts/VTokenInterfaces.sol#L53](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/VTokenInterfaces.sol#L53)  
[contracts/VTokenInterfaces.sol#L56](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/VTokenInterfaces.sol#L56)  
[contracts/VTokenInterfaces.sol#L142](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/VTokenInterfaces.sol#L142)  
[contracts/WhitePaperInterestRateModel.sol#L17](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L17)  

### [L-08] Import declarations should import specific identifiers, rather than the whole file

Using import declarations of the `form import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.  

[contracts/BaseJumpRateModelV2.sol#L4](https://github.com/code-423n4/2023-05-Venus/blob/main/contracts/BaseJumpRateModelV2.sol#L4)  
```
import "@venusprotocol/governance-contracts/contracts/Governance/IAccessControlManagerV8.sol";
```
There are 113 instances of this issue.
