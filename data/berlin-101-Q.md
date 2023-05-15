## Summary statement with observations on the codebase as a whole and opportunities for improved security practices:

After a thorough review of the code base, several findings were made regarding the following aspects:

- Used external libraries should be checked to be up-to-date and not include known vulnerabilities
- Known and new attack vectors for projects based on Compound should be reviewed and mitigated where possible
- Several typos and inconsistencies in documentation and code can be improved to reduce ambiguity and therefore increase security

In the following, the findings are listed. Non-critical findings are grouped by each smart contract.

## Low Severity Findings

### **[L-01] Venus Protocol is susceptible to a "toxic liquidation spiral" issue that was used in 0VIX exploit on April 28, 2023**

Since in the personal communication with the Venus Protocol team during the audit it was mentioned that they "would like to be made aware of general Compound issues", this exploit may be interesting to report. It may gain weight since it was just recently discovered and the knowledge may not yet have spread in the community.

The post-mortem and used steps for recovery can be read here: https://0vixprotocol.medium.com/0vix-exploit-post-mortem-15c882dcf479

### **[L-02] sweepToken function in VToken.sol is susceptible to tokens with multiple entry points**

If a token has multiple entry points the function can sweep the underlying balance circumventing a require statement.
This is a known Compound issue. See https://medium.com/chainsecurity/trueusd-compound-vulnerability-bc5b696d29e2 for the background. As the logic in Venus Protocol restricts calling the `sweepToken` function to the owner the risk is deemed low. Nevertheless it may be worthwhile to implement the changes suggested in https://medium.com/chainsecurity/trueusd-compound-vulnerability-bc5b696d29e2.

### **[L-3] Used OpenZeppelin libraries may contain a DOS issue related to proxies**

According to the `package.json` of the project OpenZeppelin libraries of version 0.8.0 or higher are used: `"@openzeppelin/contracts": "^4.8.0"`. There seems to be an existing DOS (Denial Of Service) issue related to proxies affecting OpenZeppelin contracts in versions >=3.2.0 <4.8.3 (See; https://security.snyk.io/vuln/SNYK-JS-OPENZEPPELINCONTRACTS-5425827). Since it is unclear which exact version of these contracts will be used for Venus Protocol it is encouraged to double-check that a version >= 4.8.3 is used to mitigate the mentioned issue.

### **[L-04] Pool metadata is not set when deploying a new Venus pool but it is used in PoolLens directly**

When a a new pool is deployed the metadata of the pool is not set. https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L213. But after pool creation, it is already retrievable via the `getPoolDataFromVenusPool` function in the `PoolLens.sol` (See: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L335).

The only way of setting the metadata for a pool is via the `updatePoolMetadata` function in `PoolRegistry.sol (See: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L343).

This is a potential oversight if the deployment process of a pool is not executed in a way that immediately after pool deployment the metadata is set via the respective setter function.

## Non-critical findings

### **BaseJumpRateModelV2.sol**

- **Typo:** "multiplierPerBlock" -> "multiplier per block" (to be consistent with docs above)
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L36
- **Unclear what "wrt" means in the comment:**
  The word "wrt" is only used in this file. May need more explanation:
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L60, https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L82

### **Comptroller.sol**

- **Wrong equality or documentation out of sync:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L629
  The documentation states "Callable only if the collateral is less than a predefined threshold" but the function does also not revert when the collateral is equal: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L646. If the logic is correct the documentation should be updated to "callable only if the collateral is less than or equal to a predefined threshold".

- **Line-length violated the suggested max line length of the Solidity style guide:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L827 and https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L853 exceed the suggested max line length of 120 characters (See: https://docs.soliditylang.org/en/v0.8.17/style-guide.html for style guide).

- **-1 used in documentation to indicate max integer:**
  E.g. in https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L829 "-1" is used to indicate max integer value (type(uint256).max). Since a value of "-1" may be correct for Web3 clients calling this function from the pure Solidity perspective this is not correct. Especially since Solidity version 0.8.0 that does reverts on integer underflows. This is also done in other files (`VTokens.sol`, `Comptroller.sol`, `MockPancakeSwap.sol` but only mentioned here. Find occurrences via: https://github.com/search?q=repo%3Acode-423n4%2F2023-05-venus%20-1&type=code).

- **Inline comment could also hint at healAccount function:**
  In https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L656 it is hinted that the `healBorrow` function should be used. Since the executing function is `liquidateAccount` which affects a full account the comment could be changed/extended to also hint at the `healAccount` function which also focuses on a full account: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L578

- **"Market method" term is only used once:**
  In https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1134 the term "marker method" is used. It cannot be found in any other file in the project. It either should be "market method" (which also would be the only use in the project) or be removed/replaced by a term that is common to the project and used across multiple files.

- **Different style of commenting in the same file:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1419 uses `///` style for a single line comment whereas all other functions (also of internal visibility) use multiline comments. The comments for the `expectedSender` and `_checkActionPauseState` functions should be updated to multiline comments and should potentially be extended to hold more information.

### **ComptrollerStorage.sol**

- **Comment of liquidationIncentiveMantissa state variable should use "bonus" instead of "discount":**
  In https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L67 the word "bonus" instead of "discount" is used which is misleading. See https://solodit.xyz/issues/11832 for a reference of this finding.

- **Comment of borrowCaps state variable is slightly misleading:**
  In the comment for https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L85 the word "restricts" is used ("...Defaults to zero which restricts borrowing."). This may indicate that a value of 0 does not 100% prevent borrowing as it is just restricted. To make this more clear the word "prevents" should be used instead ("...Defaults to zero which restricts borrowing.").

- **Typo:**
  The comment for the `supplCaps` state variable in https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L91 ends with "Defaults to zero which corresponds to minting notAllowed". It should end with "Defaults to zero which corresponds to minting not allowed" instead.

### **ErrorReporter.sol**

- **Inconsistency in naming errors related to freshness:**
  Starting with line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol#L46 the errors related to "freshness" end with "FreshCheck()" whereas above that line the errors ended with "FreshnessCheck()". This is inconsistent. All freshness-related errors should have the same ending. It is recommended to use "FreshnessCheck()" since this is the majority of the errors in the file and it also reads slightly better and is semantically more correct.

- **Potentially unnecessary empty code block:**
  An empty code block is used within functions for fixing "stack too deep" issues in Solidity. But it is unclear why it was added here: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L20. Should be removed if not necessary.

### **MaxLoopsLimitHelper.sol**

- **Naming of max loops limit not used consistently in comments:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L15 "max loops limit" is used whereas for example in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L35 "maxLoopsLimit" is used. The naming in comments should be consistent in the whole file.

- **Incorrect casing for max loops limit function parameter:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L16 "newmaxLoopsLimit" is used. The correct camel-case notation would be "newMaxLoopsLimit".

- **Incorrect/bad grammar in comments for \_setMaxLoopsLimit function**
  The @notice and @param listed for the `_setMaxLoopsLimit` function bother have incorrect/bad grammar in
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L22 and the following line. @notice should be "Set the limit for the loops that can be iterated to avoid the DOS". The @param should be "Limit for the max loops that can be executed at a time".

- **Typo and incorrect/bad grammar in comments for \_ensureMaxLoops function**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L35 it should be "Compare the maxLoopsLimit..." instead of "Comapre the maxLoopsLimit...".
  Also, the @notice has incorrect/bad grammar. It should be "Compare the maxLoopsLimit with number of the times the loop iterates" OR "Compare the maxLoopsLimit with number of the times loops iterate"

### **VToken.sol**

- **Typo:**
  "minter. minter" should be "minter. Minter" in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L188

- **Unusual error text for errors in sweepToken function:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L525 and https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L526 use "VToken::sweepToken:" as prefix for the error text in require statements. Across the project, no other file could be identified that uses this style of error text which makes it inconsistent.

- **It would be meaningful to also log the amount of swept tokens in SweepToken event**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol# only the contract address of the swept token is logged. It could be relevant to also log the amount of swept tokens since for events that log moving assets this is common practice. The SweepToken event would need to be extended.

- **Typo:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L620 "tokens additional tokens" is used where it should only be "additional tokens".

- **Comments for increaseAllowance and decreaseAllowance functions should also carry a warning for the related frontrunning attack:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L125 the warning for a frontrunning attack related to an allowance change is included. The functions `increaseAllowance` and `decreaseAllowance` are susceptible to the same stack but do not carry this warning. Furthermore it is recommended to use `safeIncreaseAllowance` and `safeDecreaseAllowance` functions from https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol to mitigate this issue.

- **Inconsistent approach used to implement increaseAllowance and decreaseAllowance functions:**
  In `decreaseAllowance` a require and unchecked math is used: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L650. For `increaseAllowance` this approach is not used: https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L630. This could be an inconsistency since both functions manipulate allowance up or down.

  Also `decreaseAllowance` works with a local variable name "currentAllowance" whereas `increaseAllowance` uses a local variable named "newAllowance" which serves the same purpose. Use the same naming for the local variable to create more consistency between these functions.

- **Typo:**
  In https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L640 "tokens tokens" is used where it should only be "tokens".

- **Potentially convert check for zero tokens in \_redeemFresh function into an invariant**
  It seems like the condition checked in https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L836 is more of an invariant than something that ever could happen through user input. Consider converting this into an invariant check using "assert".

  Also, the related comment in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L836 should be rewritten since it is grammatically incorrect. It should rather be "Require tokens is zero when amount is zero".

- **Unusual error text for errors in \_liquidateBorrowFresh function:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1072 and https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1075 a style of error messages (uppercase notation) is used that cannot be found in other files of the project. This may be an inconsistency.

- **Typos:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1151 "(\*requires fresh interest accrual)" is used which should be "(requires fresh interest accrual)". Same issue in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1239. But in that line, the comment should also "Updates the..." instead of "updates the..." (start uppercase).

### **VTokenInterfaces.sol**

- **Typo:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L260 it uses "the healing the" where it should be "healing the" instead.

### **PoolLens.sol**

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L111 and Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L245 should use "USD" instead of "usd".

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L385 should use "addresses" instead of "Addresses".

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L134 and other lines use "Pool" where "pool" should be used.

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L216 should not end with a ".".
- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L306 should use "VenusPool object" instead of "VenusPool Object".

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L120 should use "VToken addresses" instead of "VToken Address". Appears in multiple other lines as well.

- **Missing documentation of \_calculateNotDistributedAwards function:**
  See line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L412. All other functions are documented in the file.

### **PoolRegistry.sol**

- **Missing documentation of shortfall\_ parameter of initialize function:**
  The `initialize` function documents all passed parameters but not "shortfall" in https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L164. The parameter should be added to the docs.

- **Typos:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L203 uses "adds to the directory" where it should use "adds it to the directory". Also in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L209 it should be "The maximum limit that loops can iterate." instead of "The maximum limit for the loops can iterate."

- **Missing documentation of minLiquidatableCollateral\_ parameter of createRegistryPool function:**
  The `minLiquidatableCollateral` function documents all passed parameters but not "minLiquidatableCollateral" in https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L203. The parameter should be added to the docs.

- **Wrong positioned comment:**
  The comment in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L247 should be above line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L236.

- **Comments missing for getVTokenForAsset and getPoolsSupportedByAsset functions:**
  All other functions in the file are documented. These 2 functions should also receive documentation. Both are external functions without access restrictions.

### **PoolRegistryInterface.sol**

- **All functions should have Natspec style comments:**
  All functions below line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistryInterface.sol#L26 have comments that are not Natspec. All functions above are Natspec. For consistency reasons, all functions in the file should have the same comment style.

### **ProtocolShareReserve.sol**

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#LL62C1-L62C1 uses "@param asset Asset to be released" where it should be "@param asset Asset to be released" (whitespace).

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L94 uses "@param comptroller Comptroller address(pool)" where it should use "@param comptroller Comptroller address(pool)" (whitespace).

### **ReserveHelpers.sol**

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol uses "combined(for all pools)." where it should be "combined (for all pools)" (whitespace).

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L16 uses "Comptroller(pool) -> Asset -> amount" where it should use "Comptroller(pool) -> asset -> amount" (asset is no smart contract).

- **Typos:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L33 uses "Amount" where it should be "amount" and https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L34 uses "@param comptroller Comptroller address(pool)." where it should be "@param comptroller Comptroller address (pool)." (whitespace)

### **RiskFund.sol**

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L171 uses "comptroller doesn't exist pool registry" where it should be "comptroller doesn't exist in pool in registry" instead.

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L188 uses "Number reserved tokens." where it should be "Number of reserved tokens."

- **Typos:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L202 should be "Set the limit the loops can iterate to avoid the DOS" instead of "Set the limit for the loops can iterate to avoid the DOS". And in the next line, it should be "Limit for the max loops that can execute at a time" instead of "Limit for the max loops can execute at a time".

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L211 should be" @param comptroller Comptroller address (pool)." instead of "@param comptroller Comptroller address(pool).".

- **Typo:**
  Line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L232 uses "should" where the line above uses "must". Both lines should use "must" as they are hard criteria.

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L256 uses "finally path" where it should be "final path" or "last pass".

### **Shortfall.sol**

- **Variable naming does semantically not cover both auction scenarios:**
  In line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L40 and below the semantics of "highest" are used whereas the auction can be done in 2 modes (LARGE_RISK_FUND, LARGE_POOL_DEBT). One has the highest and the other the lowest bidder as the auction winner. Potentially change the variable naming here to something more neutral that covers both auction types. Maybe rather use "previous", "current" or "accepted" instead of "highest".

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L62 uses "next bidder. initially" where it should be "next bidder. Initially". Same issue in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L65.

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L68 uses "base asset contract" where it should be "base Asset contract"

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L74 uses "a auction" where "an auction" should be used. The same issue is present in multiple lines of the file.

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L62 uses "bidder. initially waits for 10 blocks" where it should use "bidder. Initially waits for 10 blocks.". Similar issue in line "https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L65".

- **Comments for placeBid function should reflect both auction type:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L153 only reflects the auction type where the next bid to be accepted needs to be higher as it states "greater than the previous" in the comment. The comment of the `placeBid` function should reflect both auction types.

- **Error message for invalid bid only reflects auction where next bid needs to be higher:**
  The logic block of line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L165 until https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L170 could be split into 2 blocks and each reverting with its own error message matching the auction type ("your bid is not the highest", "your bid is not the lowest").
- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L161 should use "ongoing" instead of "ongoing". Same issue with multiple other lines of the file.

- **Typo:**
  The error message in line https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L279 could be cleaner. E.g. use "you need to wait for more time to restart".

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L289 should use "@param \_nextBidderBlockLimit New next bidder" instead of "@param \_nextBidderBlockLimit New next bidder" (whitespace).

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L317 uses "BUSD" but should use "USD".

- **Typo:**
  https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L262 uses "a auction" but should use "an auction". Issue is present in multiple lines of the file.
