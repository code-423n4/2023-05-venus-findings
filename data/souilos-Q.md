# VULN 1 

## [LOW] Use safeTransferOwnership instead of transferOwnership function
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 245 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        comptrollerProxy.transferOwnership(msg.sender);

------------------------------------------------------------------------ 

### Mitigation 

Use a 2 structure transferOwnership which is safer. safeTransferOwnership, use it is more secure due to 2-stage ownership transfer. https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol










# VULN 2 

## [LOW] safeApprove of openZeppelin is deprecated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 258 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);


Found in line 259 at contests/venusContest/contracts/RiskFund/RiskFund.sol:

                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);


Found in line 322 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        token.safeApprove(address(vToken), 0);


Found in line 323 at contests/venusContest/contracts/Pool/PoolRegistry.sol:

        token.safeApprove(address(vToken), amountToSupply);

------------------------------------------------------------------------ 

### Mitigation 

Deprecated, its usage is discouraged. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20. Whenever possible, use safeIncreaseAllowance and safeDecreaseAllowance instead. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256- & https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-
