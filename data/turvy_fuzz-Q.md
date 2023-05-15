### wrong revert message "RiskFund: finally path must be convertible base asset"
wrong error message could be shown when the caller calls swapPoolsAssets with incorrect arguments for the path[][] parameter.
the error message shown: **"finally path must be convertible base asset"** is wrong and could be misleading to the users
**https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L256**
**Recommendation:**
Change to **"final path must be convertible base asset"**