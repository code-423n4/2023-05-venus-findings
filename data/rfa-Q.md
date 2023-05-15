## [QA-1] Usage of `revert()`

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L225

assertions should only be used for testing, development, and debugging purposes. assert is not meant to be used in production code. I recommend to use `require()` instead of `assert()`

## [QA-2] Arrays param length must be checked

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L885

if one of the param (`marketList` or `actionList`) has more length than the other one, the function won't revert. We can avoid this behavior by checking user input to avoid unexpected behavior