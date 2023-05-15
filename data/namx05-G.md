# 1. **Use solidity version 0.8.19 to gain some gas boost**

**Upgrade to the latest solidity version 0.8.19 to get additional gas savings.**

### **Proof Of Concept**

**All in-scope contracts.**

**See latest release for reference: [https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/](https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/)**

---

# 2. Optimize Names to Save Gas

**Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.**

**See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).**

### **Proof Of Concept**

**All in-scope contracts.**

### **Recommended Mitigation Steps**

**Find a lower method ID name for the most called functions for example `Call()` vs. `Call1()` is cheaper by 22 gas.**