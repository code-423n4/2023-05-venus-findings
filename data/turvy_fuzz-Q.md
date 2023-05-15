### wrong revert message "RiskFund: finally path must be convertible base asset"
wrong error message could be shown when the caller calls swapPoolsAssets with incorrect arguments for the path[][] parameter.
the error message shown: **"finally path must be convertible base asset"** is wrong and could be misleading to the users
**https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L256**
**Recommendation:**
Change to **"final path must be convertible base asset"**

###  Refactoring suggestion to emit update event right before updating state
emit update event right before updating state to avoid extra code for memory declaration and allocation above just for emitting event
i.e **instead of:**
 `uint256 oldMinAmountToConvert = minAmountToConvert;`
 `minAmountToConvert = minAmountToConvert_;`
 `emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);`
**Suggested:**
 `emit MinAmountToConvertUpdated(minAmountToConvert, minAmountToConvert_);`
 `minAmountToConvert = minAmountToConvert_;
`

Multiple instances of this:
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L55
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L117
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L128
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L140