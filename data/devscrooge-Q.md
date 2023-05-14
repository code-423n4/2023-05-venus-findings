# [L-01] DOS when block height reaches 2^32

## Lines of Code
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L126
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L433
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L461

## Vulnerability Details

### Description
Some functions inside the `RewardsDistributor.sol` contract will not work after the block.number of the deployed chain (in this case, the BSC) reaches a value of 2^32.

### Proof of concept
The functions `_updateRewardTokenBorrowIndex`, `_updateRewardTokenSupplyIndex` and `initializeMarket` implements the following code:

```bash=
uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");
```

The function `safe32` implements:

```bash=
function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
        require(n < 2**32, errorMessage);
        return uint32(n);
    }
```

This means that if the block height of the BSC reaches 2^32, the function execution will revert. The value of `block.number` at this moment is `28203745`. 
`2^32` is equal to `4294967296`, so condiering the current block height (`28203751`), it will be necessary that a long time passes but its mathematically possible.


### Tools Used
Manual review

### Recommended Mitigation Steps
Do not downcast to `uint32`

# [L-02] No reason for adding reserves

## Lines of Code
- https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L356-L359

## Vulnerability Details

### Description
Users can transfer undelying tokens into `VToken` contract by calling `addReserves` but there is not benefit for doing it. They will earn nothing from providing reserves to the contract.

### Impact
As there is no benefit from depositing liquidity, any user wont do it so the pools will remain with a low liquidity balance.

### Tools Used
Manual review

### Recommended Mitigation Steps
Implement a mechanism for the users that have added liquidity to gain some earnings.

