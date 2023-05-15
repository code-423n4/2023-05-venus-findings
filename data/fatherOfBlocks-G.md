**RiskFund/RiskFund.sol**
- L47 - The AmountOutMinUpdated event is created, but it is never used, therefore it generates unnecessary expense in the deploy.

- L83 - When the MaxLoopsLimit contract is initialized it is zero, therefore, in the MaxLoopsLimitHelper contract,
Line 26 validates this: require(input > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit"); therefore it is not necessary to validate two
times the same operation.

- L100/101/102/103/117/118/119/128/129/130/140/141/142 - Gas could be saved if instead of implementing it like this, it was done like this:
example:	
	require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
        emit PoolRegistryUpdated(poolRegistry, _poolRegistry);
	poolRegistry = _poolRegistry;


**Pool/PoolRegistry.sol**
- L425/426/427/434/435/436 - Gas could be saved if instead of implementing it like this, it was done like this:

	emit NewShortfallContract(shortfall, address(shortfall_));
	shortfall = shortfall_;
        

**Lens/PoolLens.sol**
- L164/165/179/180/193/194/512/513/532/533 - It is not necessary to create a variable in memory if it does not help to better understand the code and it is only used once, therefore at least expensive would be to use the operation directly where it is needed.


**Comptroller.sol**
- L707/708/709/785/788/791/915/916/917/964/965/966 - Gas could be saved if instead of implementing it like this, it was done like this:
For example:
	emit NewPriceOracle(oracle, newOracle);
	oracle = newOracle;

- L1059/1061 - It is not necessary to create a variable in memory if it does not help to understand the code better and it is only used once, therefore the least expensive would be to use the operation directly where it is needed.

- L1214 - The calculation of the allMarkets length is performed for the second time, this is an unnecessary expense of gas, since the only line that is added after the operation in line 1206 is the push in L1214. Therefore you could directly execute _ensureMaxLoops(marketsCount + 1);


**ErrorReporter.sol**
GO
- L7/9/10/12/15/19/23/29/32/37/39/42/51 - Multiple errors are created in the contract and are never used, therefore they should be eliminated, since they generate a unnecessary expense in the deploy.


**MaxLoopsLimitHelper.sol**
GO
- L28/29/31 - Gas could be saved if instead of implementing it like this, it was done like this:
example:	
	emit MaxLoopsLimitUpdated(maxLoopsLimit, limit);
	maxLoopsLimit = limit;


**ExponentialNoError.sol**
- L22/23 - Two constants are created: halfExpScale and mantissaOne in storage, but they are never used, therefore it is an unnecessary expense in the deploy.

- L39/40 - It is not necessary to create a variable in memory if it does not help to better understand the code and it is only used once, therefore the least expensive would be to use the operation directly where it is needed.


**RiskFund/ProtocolShareReserve.sol**
- L55/56/57 - Gas could be saved if instead of implementing it like this, it was done like this:
example:	
	require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");
	emit PoolRegistryUpdated(poolRegistry;, _poolRegistry);
        poolRegistry = _poolRegistry;
