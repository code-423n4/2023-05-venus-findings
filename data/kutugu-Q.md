# Findings Summary

| ID     | Title                                                       | Severity |
| ------ | ----------------------------------------------------------- | -------- |
| [L-01] | EnterMarkets prevents users from entering duplicate markets | Low      |
| [L-02] | DecreaseAllowance can be fail by frontrun                   | Low      |
| [L-03] | VToken mint can be sandwiched                               | Low      |

# Detailed Findings

# [L-01] EnterMarkets prevents users from entering duplicate markets

## Link

[enterMarkets](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L159)   

## Description

The user's second enterMarket list may duplicate the first one, which can cause `_ensureMaxLoops` to revert the second `enterMarket` execution.    

## Poc

```
MaxLoopsLimit: 5
Firstly enterMarket, vTokens: [USDC, DAI, WETH]
Secondly enterMarket, vTokens: [USDC, WBNB, WBTC]

The second enterMarket execution failed, even though the total number of markets the user actually entered was 5
```

## Recommendations

Perform [_addToMarket](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L161-L167) first, then `_ensureMaxLoops`. `_addToMarket` will [ignore](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L1186) the market already entered.    

# [L-02] DecreaseAllowance can be fail by frontrun

## Link

[decreaseAllowance](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L650)   

## Description

The `decreaseAllowance` revert if the current allowance is less than the allowance to be reduced.    
Malicious users can frontrun `decreaseAllowance` operation, spend part of the token, cause the overall call revert.    

## Poc

```
UserA approve userB 1 ether allowance
UserA want to reduce userB 1 ether allowance
UserB just need spend 1wei allowance, revert userA's call
```

## Recommendations

If the current allowance is less than the allowance to be reduced, set allowance to 0. 

# [L-03] VToken mint can be sandwiched

## Link

[_mintFresh](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L776)   

## Description

[_exchangeRateStored](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L1463) relies on the balance of `underlying`. Similar to `ERC4626`, mint can be sandwiched.     
Although [here](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L776) the calculation uses scaling to make the attack cost amplified `expScale` times.      
By contrast, [redeem](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/VToken.sol#L837) limits the redeem amount to be 0.     
 
## Poc

```
UserA want to mint VToken
Attacker frontrun mint，ratio is initialExchangeRateMantissa
Attacker transfer huge underlying token to VToken directly, amplify _exchangeRateStored
UserA mint, because _exchangeRateStored is huge, the mint amount to be 0
Attacker redeem all VToken, including userA's underlying Token.
```

## Recommendations

Limit the mint amount not to be 0.  
