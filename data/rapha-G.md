# Description
Calldata arrays are more gas efficient compared to memory arrays when passed as function arguments. That is true when the following conditions have been met:

1. The function argument is read-only
2. The function is not explicitly receiving a memory array.

The following files are affected:
```solidity
File: contracts/Comptroller.sol
154:   function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {
```
https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Comptroller.sol#L154
This actually is in contrast to the extended `ComptrollerInterface.sol` which implements `function enterMarkets(address[] calldata)`.

```solidity
File: contracts/Rewards/RewardsDistributor.sol
197:    function setRewardTokenSpeeds(
198:        VToken[] memory vTokens,
199:        uint256[] memory supplySpeeds,
200:        uint256[] memory borrowSpeeds
201:    ) external {
```
https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/Rewards/RewardsDistributor.sol#L197

# Impact
The total gas saved will amount to **952**
# Remediation
Change all the memory arrays received in the function arguments to calldata arrays:
```solidity
File: contracts/Comptroller.sol
154:   function enterMarkets(address[] calldata vTokens) external override returns (uint256[] memory) {
```

```solidity
File: contracts/Rewards/RewardsDistributor.sol
197:    function setRewardTokenSpeeds(
198:        VToken[] calldata vTokens,
199:        uint256[] calldata supplySpeeds,
200:        uint256[] calldata borrowSpeeds
201:    ) external {
```


