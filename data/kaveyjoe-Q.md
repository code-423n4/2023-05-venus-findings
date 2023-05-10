Bug report: issue with the setShortfallContractAddress function

Target : https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

There was a issue with the setShortfallContractAddress function that sets the address of the shortfall contract. The function checks if the address of the shortfallContractAddress_ is not equal to zero and then proceeds to check if the convertibleBaseAsset of the new shortfallContractAddress_ matches with the one set in the contract. However, the check is not safe, and it can be bypassed by an attacker.

An attacker can create a new shortfallContractAddress_ with a different convertibleBaseAsset and then use the setShortfallContractAddress function to set it as the new shortfall contract. Since the shortfall contract is used in other parts of the contract, the attacker can exploit this issue to execute a malicious action.

To fix this issue, the function should also check if the sender is the owner of the contract. Only the owner should be allowed to change the shortfall contract address. The check can be added by adding the onlyOwner modifier to the function.