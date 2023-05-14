 
# Gas Optimizations Report

This report focuses on Venus Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Venus protocol are categorised into 10 main areas; with further multiple instances in each of the category.


# [G-01] Use solidity version 0.8.19 to gain some gas boost (28 Instances)

Use latest version of solidity i.e., 0.8.19 for getting additional gas saving for reference check this article.

Link to the code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerInterface.sol
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/JumpRateModelFactory.sol
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistryInterface.sol
21.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol
22.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol
23.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IRiskFund.sol
24.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/IPancakeswapV2Router.sol
25.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/WhitePaperInterestRateModelFactory.sol
26.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol
27.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IProtocolShareReserve.sol
28.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/IShortfall.sol




# [G-02] Immutable has more gas efficiency than constant (09 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.


Link to the Code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L60
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L29
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L53
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L56
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L142
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L20
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L21
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L23


# [G-03] Use hardcode address instead address(this) (24 Instances)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

Link to the code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L501
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L504
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L527
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L749
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L842
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L879
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L936
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1027
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1068
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1078
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1079
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1104
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1274
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1275
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1276
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1304
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1423
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L187
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L193
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L417
21.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L415
22.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L416
23.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L417
24.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L264



# [G-04] 0perator assignment is more gas efficient than compound assignment (03 Instance)

Compound assignment operators (+= / -=) are more expensive in terms of gas consumption and needs to be avoided.

Operator assignments (a = a + b / a - b) are preferable in terms of gas optimization.

Link to the Code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L630
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L249
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L250


# [G-05] Use != 0 instead of > 0 for unsigned integer comparison (20 Instances)
	
Link to the Code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L367
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1262
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L818
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1369
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L466
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L470
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L483
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L486
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L490
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L418
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L435
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L438
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L446
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L463
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L466
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L474



# [G-06] Use assembly to check for address(0) (25 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L962
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L72
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L134
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L196
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L626
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L648
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1399
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1408
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L137
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L138
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L166
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L167
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L169
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L170
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L180
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L189
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L214
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L349
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L468
21.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L225
22.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L226
23.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L257
24.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80
25.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L81

# [G-07] Cache array length outside of loop (15 Instances)

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

Link to the Code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L162
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L274
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L304
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L376
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L402
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L521
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L558
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L584
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L611
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L669
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L690
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L819
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L127
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L391
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L209


# [G-08] Use calldata instead of memory for function parameters (27 Instances)

Using calldata in external function does not require data to be stored, which reduced the process time as compared to memory. This in return saves gas during calling the data.

Link to the Code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L64
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L65
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1355
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1356
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1360
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L388
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L414
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L456
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L457
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L478
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L499
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L500
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L520
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L187
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L197
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L277
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L374
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L458
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L255
21.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L343
22.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L354
23.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L29
24.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L38
25.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L47
26.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L59
27.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L145



# [G-09] Public functions to external instead (03 Instances)

Functions with public visibility, if not called within the contract needed to be changed external.

Link to the Code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L112
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L67
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L49


# [G-10] Use uint256(1)/uint256(2) instead for true and false boolean states (05 Instances)

Boolean for storage if not used, this avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L811
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L944
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1194
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L37
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1394
