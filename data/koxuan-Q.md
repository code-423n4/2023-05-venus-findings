## Low Risk Issues

### [L&#x2011;01] ERC20 tokens that does not implement optional decimals method cannot be used

Underlying token that does not implement optional decimals method cannot be used

> [EIP-20](https://eips.ethereum.org/EIPS/eip-20#decimals) OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present.

*There are 2 instances of this issue:*

```solidity
File: contracts/Pool/PoolRegistry.sol
        uint256 underlyingDecimals = IERC20Metadata(input.asset).decimals();
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L284


```solidity
File: contracts/Lens/PoolLens.sol
        underlyingDecimals = IERC20Metadata(vToken.underlying()).decimals();
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L361

## Non-critical Issues

### [N&#x2011;01] For functions, follow Solidity standard naming conventions

internal and private functions: the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

*There are 4 instances of this issue:*

```solidity
File: contracts/Lens/PoolLens.sol

    function updateMarketBorrowIndex(
        address vToken,
        RewardsDistributor rewardsDistributor,
        RewardTokenState memory borrowState,
        Exp memory marketBorrowIndex
    ) internal view {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L453-L458

```solidity
File: contracts/Lens/PoolLens.sol

    function updateMarketSupplyIndex(
        address vToken,
        RewardsDistributor rewardsDistributor,
        RewardTokenState memory supplyState
    ) internal view {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L475-L479

```solidity
File: contracts/Lens/PoolLens.sol

    function calculateBorrowerReward(
        address vToken,
        RewardsDistributor rewardsDistributor,
        address borrower,
        RewardTokenState memory borrowState,
        Exp memory marketBorrowIndex
    ) internal view returns (uint256) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L495-L501

```solidity
File: contracts/Lens/PoolLens.sol

    function calculateSupplierReward(
        address vToken,
        RewardsDistributor rewardsDistributor,
        address supplier,
        RewardTokenState memory supplyState
    ) internal view returns (uint256) {
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L516-L521

### [N&#x2011;02] Include return parameters for Natspec comments

Recommend to add a `@return` tag for functions that has any return value


*There is 1 instance of this issue:*

```solidity
File: contracts/Comptroller.sol

    /**
     * @notice A marker method that returns true for a valid Comptroller contract
     */
    function isComptroller() external pure override returns (bool) {
        return _isComptroller;
    }
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1133-L1138

### [N&#x2011;03] immutable should be uppercase

*There is 3 instance of this issue:*

```solidity
File: contracts/Comptroller.sol

    address public immutable poolRegistry;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L27

```solidity
File: contracts/WhitePaperInterestRateModel.sol

    uint256 public immutable multiplierPerBlock;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L22

```solidity
File: contracts/WhitePaperInterestRateModel.sol

    uint256 public immutable baseRatePerBlock;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L27


### [N&#x2011;04] Missing NatSpec

*There are 2 instances of this issue:*

```solidity
File: contracts/Factories/VTokenProxyFactory.sol

    event VTokenProxyDeployed(VTokenArgs args);

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L26

```solidity
File: contracts/Factories/VTokenProxyFactory.sol

    function deployVTokenProxy(VTokenArgs memory input) external returns (VToken) {
    
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L28
