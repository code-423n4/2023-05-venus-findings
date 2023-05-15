| ID     |                           Title                           | Instances |
| :----- | :-------------------------------------------------------: | --------: |
| [G-01] |            Use Assembly for `address(0)` check            |        49 |
| [G-02] | Storing variable`.length` in local variables in for loops |         3 |
| [G-03] |           Write the result rather than exponent           |         3 |
| [G-04] |  Use Assembly in the constructor while assigning address  |         5 |
| [G-05] |    Use `a=a+b` or `a=a-b` instead of `a+=b` or `a+=b`     |        10 |
| [G-06] | Use `bytes32` instead of short`string` datatype variable  |         2 |
| [G-07] |        Use `!=` instead of `>0` for uint datatypes        |         6 |

### [G-01] Use Assembly for `address(0)` check

We can save `6 gas` by writing the `address(0)` check of the require statement in assembly.
For Eg:

Instead of this...

```solidity

function ownerNotZero(address _addr) public pure {
    require(_addr != address(0), "zero address)");
}
```

We can use this...

```solidity

function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}
```

```solidity
BaseJumpRateModelV2.sol

require(address(accessControlManager_) != address(0), "invalid ACM address");

Comptroller.sol

require(poolRegistry_ != address(0), "invalid pool registry address");
```

### [G-02] Storing variable`.length` in local variables in for loops

Use an local variable to store the length `uint256 length = rewardsDistributors.length`so that for loop doesn't the fetch the length all time.

```solidity
PoolLens.sol

for (uint256 i; i < rewardsDistributors.length; ++i) {...}

 for (uint256 i; i < markets.length; ++i) {...}
```

### [G-03] Write the result rather than exponent

Use
For Example:
`uint256 var = 1000` rather than exponentiation `uint256 var = 10**3`

### [G-04] Use Assembly in the constructor while assigning address

Use assembly to assign `address` in the `constructor()` to save Gas.

```solidity
    // @audit check all the contructor assigning address to use assembly -> it will optimize it
    constructor(address poolRegistry_) {
        require(poolRegistry_ != address(0), "invalid pool registry address");
        // @audit
        poolRegistry = poolRegistry_; //
        @note use assembly to store address in constructor
        // @note eg->  assembly {sstore(vault. revenueAddress, _revenueAddress)}
        _disableInitializers();
    }
```

### [G-05] Use `a=a+b` or `a=a-b` instead of `a+=b` or `a+=b`

Use `a=a+b` or `a=a-b` instead of `a+=b` or `a+=b` which saves Gas.

```solidity
ProtocolShareReserve.sol

 assetsReserves[asset] -= amount; // @note

poolsAssetsReserves[comptroller][asset]
-= amount; // @note
```

### [G-06] Use `bytes32` instead of short`string` datatype variable

Use `bytes32` instead of `string` for short strings to save Gas.

```solidity
VTokenInterfaces.sol

 string public name;

 string public symbol;
```

### [G-07] Use `!=` instead of `>0` for uint datatypes

As here `deletaBlocks` has datatype `uint256` and as it cannot be negative value, it is better to use `!=0` rather than `>0`to save gas.

```solidity
RewardsDistributor.sol
       if (deltaBlocks > 0 && supplySpeed > 0) {...}

```
