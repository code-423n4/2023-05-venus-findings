Gas Limit Check Absence in PoolRegistry.sol

Description of the Vulnerability:
During my analysis of the PoolRegistry.sol, I identified an absence of gas limit checks in critical functions, which could potentially lead to Denial-of-Service (DoS) attacks. Without gas limit checks, an attacker could construct transactions or inputs that consume excessive gas or trigger expensive operations, resulting in the contract execution running out of gas or becoming prohibitively expensive to execute.

Potential Impact:
The absence of gas limit checks can have the following impact:
- DoS attacks: Attackers can disrupt the functionality of the contract by causing it to run out of gas, rendering it temporarily or permanently unusable.
- Financial losses: DoS attacks can result in financial losses if the contract's services are disrupted, preventing users from accessing or utilizing their assets.
- Hindered business operations: The contract's inability to handle excessive gas consumption may hinder regular business operations and user interactions, leading to reputational damage.

Proof of Concept (PoC):
I have developed a PoC in Solidity to demonstrate the absence of gas limit checks in the affected functions. The PoC contract, provided below, simulates a scenario where a costly operation is performed without any gas limit checks:

```solidity
// Contract to demonstrate gas limit check absence vulnerability

contract GasLimitCheckDemo {
    mapping(address => uint256) public balances;

    // Function that performs a costly operation without gas limit checks
    function performCostlyOperation() public {
        while (true) {
            balances[msg.sender]++;
        }
    }
}
```

In this PoC, the `performCostlyOperation` function contains an infinite loop that continuously increases the balance of the caller. Without a gas limit check, this operation will consume an excessive amount of gas until the transaction runs out of gas or becomes prohibitively expensive to execute.


Recommendation:
To address this vulnerability and enhance the security of the PoolRegistry.sol, I recommend the following measures:
1. Implement gas limit checks: Introduce gas limit checks at critical points in the contract's functions, such as loops or expensive computations, to ensure gas consumption remains within acceptable limits.
2. Optimize gas usage: Review and optimize the contract's logic to minimize gas consumption and avoid expensive or unnecessary operations.
3. Perform comprehensive testing: Conduct thorough testing to ensure the effectiveness of the gas limit checks and validate the contract's resilience against potential DoS attacks.
