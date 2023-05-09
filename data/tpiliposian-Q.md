## Summary
In the function `initialize()` the address `accessControlManager` lacks a zero address check.

## Code Snippet
PATH: `Comptroller.sol:initialize()` (L138-143)

```
    function initialize(uint256 loopLimit, address accessControlManager) external initializer {
        __Ownable2Step_init();
        __AccessControlled_init_unchained(accessControlManager);

        _setMaxLoopsLimit(loopLimit);
    }
```

## Recommendation
Consider adding a zero address check to the variable `accessControlManager` in the initialize() function, it would be a good practice to add a zero address check for any address parameters being passed in. This can help ensure that the contract is initialized with valid parameters and prevent potential issues.