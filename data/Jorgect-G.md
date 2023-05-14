# Gas Optimization

## [G-01] Use modifier can save gas.

the next condition in the comptroller contract  is used 4 times in the code consider create a modifier and save gas:

```
file:/contracts/Comptroller.sol
if (!markets[vToken].isListed) { 
            revert MarketNotListed(address(vToken));
        }
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L256-L258

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L333-L335

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L395-L397

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L256-L258
 
you can create the next modifier and aplied in the functions:

```
modifier isMarketListed(address vToken){

if (!markets[vToken].isListed) { 
            revert MarketNotListed(address(vToken));
        }
}
```

## [G-02] Use modifier can save gas.

the next condition in the comptroller contract  is used 4 times in the code consider create a modifier and save gas:

```
file:/contracts/Vtoken.sol
if (msg.sender != address(comptroller)) {
            revert ForceLiquidateBorrowUnauthorized();
        }
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L395-L397

https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L456-L458

 
you can create the next modifier and aplied in the functions:

```
modifier isContropller(){

if (msg.sender != address(comptroller)) {
            revert ForceLiquidateBorrowUnauthorized();
        }
}
```




