## [QA-1] Usage of `revert()`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L225

assertions should only be used for testing, development, and debugging purposes. assert is not meant to be used in production code. I recommend to use `require()` instead of `assert()`