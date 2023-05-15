# [G-01] Using private rather than public for constants, saves gas
## If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

There are 1 instances of this issue:

`23:     uint256 public constant blocksPerYear = 10512000;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L23

# [G-02] Use `assembly` to check for `address(0)`

There are 18 instances of this issue:

`File: contracts/BaseJumpRateModelV2.sol`
`72:        require(address(accessControlManager_) != address(0), "invalid ACM address");` 
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L72

`File: contracts/Comptroller.sol`
`128:        require(poolRegistry_ != address(0), "invalid pool registry address");`
`962:        require(address(newOracle) != address(0), "invalid price oracle address");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128

`File: contracts/VToken.sol`
`72:        require(admin_ != address(0), "invalid admin address");`
`134        require(spender != address(0), "invalid spender address");`
`196:        require(minter != address(0), "invalid minter address");`
`625:        require(spender != address(0), "invalid spender address");`
`646:        require(spender != address(0), "invalid spender address");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72

`File: contracts/RiskFund/ProtocolShareReserve.sol`
`40:        require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");`
`41:        require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");`
`54:require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");`
`71:        require(asset != address(0), "ProtocolShareReserve: Asset address invalid");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40

`File: contracts/RiskFund/RiskFund.sol`
`80:        require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");`
`81:       require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");`
`100:        require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");`
`111:        require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");`
`127:        require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");`
`157:        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80

# [G-03]Use hardcode address instead `address(this)`
## Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded`address`. Foundry’s `script.sol` and solmate’s `LibRlp.sol` contracts can help achieve this.

### References:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

There are 3 instances of this issue:

`File: `
`98:            revert Unauthorized(msg.sender, address(this), signature);`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L98

`File: `
`501:        if (seizerContract == address(this))`
`504:            if (address(VToken(vTokenCollateral).comptroller()) != address(this))`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L501









# [G-04] —Use an `unchecked` block when operands can’t underflow/overflow
## The risk of for `loops` getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the `arrays` are extremely long, it will take a massive amount of time and gas to let the for loop overflow.

There are 15 instances of this issue:


`File: contracts/Comptroller.sol`
`for (uint256 i; i < len; ++i) {
            VToken vToken = VToken(vTokens[i]);
            _addToMarket(vToken, msg.sender);
            results[i] = NO_ERROR;
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L162-#L167

`   for (uint256 i; i < len; ++i) {
            if (userAssetList[i] == vToken) {
                assetIndex = i;
                break;
            }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L217-#L221

`        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L274-#L277

`  for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, redeemer);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L304-#L307

`        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L376-#L379

`        for (uint256 i; i < rewardDistributorsCount; ++i) {
            Exp memory borrowIndex = Exp({ mantissa: VToken(vToken).borrowIndex() });
            rewardsDistributors[i].updateRewardTokenBorrowIndex(vToken, borrowIndex);
            rewardsDistributors[i].distributeBorrowerRewardToken(vToken, borrower, borrowIndex);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402-#L406


`  for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vTokenCollateral);
            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, borrower);
            rewardsDistributors[i].distributeSupplierRewardToken(vTokenCollateral, liquidator);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L521-#L525

`        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, src);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, dst);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L558-#L562

`        for (uint256 i; i < userAssetsCount; ++i) {
            userAssets[i].accrueInterest();
            oracle.updatePrice(address(userAssets[i]));
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L584-#L587

`        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].initializeMarket(address(vToken));
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L819-#L821


`        for (uint256 i; i < numMarkets; ++i) {
            borrowCaps[address(vTokens[i])] = newBorrowCaps[i];
            emit NewBorrowCap(vTokens[i], newBorrowCaps[i]);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L846-#L849

`        for (uint256 i; i < vTokensCount; ++i) {
            supplyCaps[address(vTokens[i])] = newSupplyCaps[i];
            emit NewSupplyCap(vTokens[i], newSupplyCaps[i]);
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L871-#L874

`       for (uint256 i; i < rewardsDistributorsLength; ++i) {
            address rewardToken = address(rewardsDistributors[i].rewardToken());
            require(
                rewardToken != address(_rewardsDistributor.rewardToken()),
                "distributor already exists with this reward"
            );
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L931-#L938

`        for (uint256 i; i < marketsCount; ++i) {
            _rewardsDistributor.initializeMarket(address(allMarkets[i]));
        }`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L948-#L950

# [G-05]  Use `constants` instead of `type(uintx).max`

There are 2 instances of this issue:
`File: contracts/Comptroller.sol`
`262:        if (supplyCap != type(uint256).max)`
`351:        if (borrowCap != type(uint256).max)`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L262

# [G-06] `require()`/`revert()` strings longer than 32 bytes cost extra gas
## Each extra memory word of bytes past the original `32` incurs an MSTORE which costs `3` gas.

There are 36 instances of this issue:

`File: contracts/Comptroller.sol`
`692:            require(borrowBalance == 0, "Nonzero borrow balance after liquidation");`
`704:        require(closeFactorMaxMantissa >= newCloseFactorMantissa, "Close factor greater than maximum close factor");`
`705:        require(closeFactorMinMantissa <= newCloseFactorMantissa, "Close factor smaller than minimum close factor");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L692
`780:      require(newLiquidationIncentiveMantissa >= 1e18, "liquidation incentive should be greater than 1e18");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L692

`File: contracts/VToken.sol`
`489:        require(msg.sender == shortfall, "only shortfall contract can update bad debt");`
`490:        require(recoveredAmount_ <= badDebt, "more than bad debt recovered from auction");`
`525:        require(msg.sender == owner(), "VToken::sweepToken: only admin can sweep tokens");`
`526:        require(address(token) != underlying, "VToken::sweepToken: can not sweep underlying token");`
`805: require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");`
`1072:        require(amountSeizeError == NO_ERROR, "LIQUIDATE_COMPTROLLER_CALCULATE_AMOUNT_SEIZE_FAILED");`
`1365:         require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L489

`File: RiskFund/ReserveHelpers.sol`
`39: require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");`
`40:       require(asset != address(0), "ReserveHelpers: Asset address invalid");`
`51:  require(ComptrollerInterface(comptroller).isComptroller(), "ReserveHelpers: Comptroller address invalid");`
`52:        require(asset != address(0), "ReserveHelpers: Asset address invalid");`
`53:        require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");`
`54:  require(
55:            PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
56:            "ReserveHelpers: The pool doesn't support the asset"
57:        );`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L39

`File: contracts/RiskFund/RiskFund.sol`
`80:        require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");`
`81:        require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");`
`82:        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");`
`83:        require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");`
`100:        require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");`
`111:        require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");`
`112:        require(
113:            IShortfall(shortfallContractAddress_).convertibleBaseAsset() == convertibleBaseAsset,
114:            "Risk Fund: Base asset doesn't match"
115:        );`
`127:        require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");`
`139:        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");`
`157:        require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");`
`158:        require(markets.length == amountsOutMin.length, "Risk fund: markets and amountsOutMin are unequal lengths");`
`159:        require(markets.length == paths.length, "Risk fund: markets and paths are unequal lengths");`
`171:            require(pool.comptroller == comptroller, "comptroller doesn't exist pool registry");`
`191:        require(msg.sender == shortfall, "Risk fund: Only callable by Shortfall contract");`
`192:        require(amount <= poolReserves[comptroller], "Risk Fund: Insufficient pool reserve.");`
`231:        require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");`
`232:        require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");`
`253:                    require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");`
`254:                    require(
255:                        path[path.length - 1] == convertibleBaseAsset,
256:                        "RiskFund: finally path must be convertible base asset"
257:                    );`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80

# [G-07] Use double `require` instead of using `&&`
## Using double `require` instead of operator `&&` can save more gas. When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.

There are 2 instances of this issue:

`File: contracts/Comptroller.sol`
`842:        require(numMarkets != 0 && numMarkets == numBorrowCaps, "invalid input");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L842

`File: contracts/VToken.sol`
`1365:        require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1365



# [G-08] Using `bools` for storage incurs overhead
## `Booleans` are more expensive than uint256 or any type that takes up a full  word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

References: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

There are 1 instances of this issue:

`File: contracts/InterestRateModel.sol`
`10:    bool public constant isInterestRateModel = true;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol#L10

# [G-09]`<x> += <y>` Costs More Gas Than ` <x> = <x> + <y>` For State Variables

## References: https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8

There are 4 instances of this issue:

`File: contracts/VToken.sol`
`630:        newAllowance += addedValue;`
`652:            currentAllowance -= subtractedValue;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L630

`File: contracts/RiskFund/ReserveHelpers.sol`
`66:            assetsReserves[asset] += balanceDifference;`
`67:            poolsAssetsReserves[comptroller][asset] += balanceDifference;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L66

# [G-10] Using private rather than public for constants, saves gas
## If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

There are 1 instances of this issue:

`File: contracts/WhitePaperInterestRateModel.sol`
`17:    uint256 public constant blocksPerYear = 2102400;`
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L17


# [G-11]Help the optimizer by Saving a storage variable’s reference instead of repeatedly fetching it
## To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.The effect can be quite significant.

As an example, instead of repeatedly calling `someMap[someIndex]`
save its reference like this: SomeStruct storage someStruct = `someMap[someIndex]` and use it.

# [G-12]Optimize names to save gas
## Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more here.
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92


## Proof Of Concept
All in-scope contracts.

Recommended Mitigation Steps
Find a lower method ID name for the most called functions for example `Call()` vs .`Call1()` is cheaper by   `22` gas.

For example, the function IDs in the `Gauge.sol` contract will be the most used; A lower method ID may be given.

# [G-13] Use solidity version ` 0.8.19` to gain some gas boost
## Upgrade to the latest solidity version `0.8.19` to get additional gas savings.

See latest release for reference:
https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/


