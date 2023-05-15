## [N-01] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

There are 6 instances of this issue in 5 files:

    File: Rewards/RewardsDistributor.sol

    26: mapping(address => mapping(address => uint256)) public rewardTokenSupplierIndex;

    50: mapping(address => mapping(address => uint256)) public rewardTokenBorrowerIndex;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

    File: Pool/PoolRegistry.sol

    103: mapping(address => mapping(address => address)) private _vTokens;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

    File: VTokenInterfaces.sol

    110: mapping(address => mapping(address => uint256)) internal transferAllowances;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol

    File: ComptrollerStorage.sol

    95: mapping(address => mapping(Action => bool)) internal _actionPaused;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol

    File: RiskFund/ReserveHelpers.sol

    17: mapping(address => mapping(address => uint256)) internal poolsAssetsReserves;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol

## [N-02] Using Vulnerable Version of Openzeppelin

The package.json configuration file says that the project is using 4.8.0 of OpenZeppelin which has a vulnerability.

Although there is no security vulnerability covering the project, it is recommended to use the latest version 4.8.3.

There is 1 instance of this issue in 1 file:

    File: package.json
 
    51: "@openzeppelin/contracts": "^4.8.0",

https://github.com/code-423n4/2023-05-venus/blob/main/package.json#L51

## [N-03] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 6 instances of this issue in 3 files:

    File: Comptroller.sol

    802: _checkSenderIs(poolRegistry);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol 

    File: VToken.sol

    395: if (msg.sender != address(comptroller)) {
    396:     revert HealBorrowUnauthorized();
    397: }

    456: if (msg.sender != address(comptroller)) {
    457:     revert ForceLiquidateBorrowUnauthorized();
    458: }

    489: require(msg.sender == shortfall, "only shortfall contract can update bad debt");

    525: require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: RiskFund/RiskFund.sol

    191: require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

## [N-04] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist

There is 1 instance of this issue in 1 file:

    File: Pool/PoolRegistry.sol

    290: 10**(underlyingDecimals + 18 - input.decimals),
  
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol 

## [N-05]  Use delete to clear variables instead of zero assignment

You can use the delete keyword instead of setting the variable as zero.

There are 7 instances of this issue in 2 files: 

    File: Comptroller.sol

    812: newMarket.collateralFactorMantissa = 0;

    813: newMarket.liquidationThresholdMantissa = 0;

    1353: snapshot.liquidity = 0;

    1355: snapshot.liquidity = 0;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: Shortfall/Shortfall.sol

    370: auction.highestBidBps = 0;

    371: auction.highestBidBlock = 0;

    376: auction.marketDebt[vToken] = 0;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol