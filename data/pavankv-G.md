## 1.Declare returns varaible locally which saves deployment gas :-

code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L354
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1368
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#LL223C41-L223C41
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L248
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L312
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L388
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L416
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L501
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L521


## 2. Change design of updating struct to save gas :-

By changing the pattern of assigning value to the structure, gas savings of ``~130`` per instance are achieved. In addition, this use will provide significant savings in distribution costs.


Old
```
Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
```

New
```
Exp memory exchangeRate;
exchangeRate.mantissa =VToken(vToken).exchangeRateStored(); 
```


code snippet:-
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L264
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L403
https://github.com/code-423n/2023-05-venus/blob/main/contracts/VToken.sol#L756
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L149
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L371
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L600
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L295
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L502
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L503


