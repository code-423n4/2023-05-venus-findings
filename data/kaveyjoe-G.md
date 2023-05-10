Target : https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol

Issue: The VTokenStorage contract imports "ErrorReporter.sol" but it is not used in the contract, which causes unnecessary gas cost for deployment and increases the contract's size.

Severity: Low

Description:

In the code, VTokenStorage contract imports "ErrorReporter.sol", but it is not used anywhere in the contract, which causes unnecessary gas cost for deployment and increases the contract's size.

Recommendation:

It is recommended to remove the import statement for "ErrorReporter.sol" from the VTokenStorage contract to reduce the gas cost for deployment and decrease the contract's size.