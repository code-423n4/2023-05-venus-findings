## Title
Do not use Deprecated Library Functions

## Links to affected code
  2023-05-venus/contracts/Pool/PoolRegistry.sol::322 => token.safeApprove(address(vToken), 0);
  2023-05-venus/contracts/Pool/PoolRegistry.sol::323 => token.safeApprove(address(vToken), amountToSupply);
  2023-05-venus/contracts/RiskFund/RiskFund.sol::258 => IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
  2023-05-venus/contracts/RiskFund/RiskFund.sol::259 => IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);

## Impact
The usage of deprecated library functions should be discouraged.

This issue is mostly related to OpenZeppelin libraries.

## Proof of Concept
🤦 Bad:

use SafeERC20 for IERC20;

// ...

IERC20(token).safeApprove(spender, value);
🚀 Good:

use SafeERC20 for IERC20;

// ...

IERC20(token).safeIncreaseAllowance(spender, value);

## Tools Used
Manual

## Recommended Mitigation Steps
🚀 Good:

use SafeERC20 for IERC20;

// ...

IERC20(token).safeIncreaseAllowance(spender, value);