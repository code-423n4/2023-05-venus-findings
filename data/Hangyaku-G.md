# Gas optimisations
##  Use Unchecked for increments in for loops
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L846
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L819
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L690
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L669
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L611
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L584
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L558
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L521
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L274
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L217

## Use Double Require instead of &&
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L842
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1365

## use double if instead of &&
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L837
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L483
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526

## use a more recent version if solidity
upgrade to a recent version of solidity to save gas
where:all contracts
## use custom errors
where :most contracts
ex:https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L163
