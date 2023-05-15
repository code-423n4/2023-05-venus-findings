## [G-01] Variables that are set only at time of construction should be set as immutable

The variables that are only set at the time of construction of contract should be set as immutable to save gas cost.

There is 1 instance of this issue in 1 file:

    File: BaseJumpRateModelV2.sol

    74: accessControlManager = accessControlManager_;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol

#### Test Contract 

    contract GasTest is DSTest {
        Contract0 c0;
        Contract_Optimized c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract_Optimized();
        }

        function testGas() public {
            c0.getNum();
            c1.getNum();
        }
    }

    contract Contract0 {
        uint256 num;

        constructor(){
            num = 10;
        }

        function getNum() public returns(uint256){
            return num;
        }
    }

    contract Contract_Optimized {
        uint256 immutable num;

        constructor(){
            num = 10;
        }

        function getNum() public returns(uint256){
            return num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 46181                                     | 154             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| getNum                                    | 2246            | 2246 | 2246   | 2246 | 1       |

| Contract_Optimized contract                        |                 |     |        |     |         |
|----------------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                                    | Deployment Size |     |        |     |         |
| 30108                                              | 195             |     |        |     |         |
| Function Name                                      | min             | avg | median | max | # calls |
| getNum                                             | 146             | 146 | 146    | 146 | 1       |

## [G-02] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 14 instances of this issue in 6 files:

    File: Comptroller.sol

    217: for (uint256 i; i < len; ++i) {

    274: for (uint256 i; i < rewardDistributorsCount; ++i) {

    304: for (uint256 i; i < rewardDistributorsCount; ++i) {

    376: for (uint256 i; i < rewardDistributorsCount; ++i) {

    402: for (uint256 i; i < rewardDistributorsCount; ++i) {

    521: for (uint256 i; i < rewardDistributorsCount; ++i) {

    558: for (uint256 i; i < rewardDistributorsCount; ++i) {

    584: for (uint256 i; i < userAssetsCount; ++i) {

    611: for (uint256 i; i < userAssetsCount; ++i) {

    669: for (uint256 i; i < ordersCount; ++i) {

    690: for (uint256 i; i < marketsCount; ++i) {

    819: for (uint256 i; i < rewardDistributorsCount; ++i) {

    846: for (uint256 i; i < numMarkets; ++i) {

    871: for (uint256 i; i < vTokensCount; ++i) {

    932: for (uint256 i; i < rewardsDistributorsLength; ++i) {

    948: for (uint256 i; i < marketsCount; ++i) {

    1122: for (uint256 i; i < rewardsDistributorsLength; ++i) {

    1208: for (uint256 i; i < marketsCount; ++i) {

    1307: for (uint256 i; i < assetsCount; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: Lens/PoolLens.sol

    127: for (uint256 i; i < vTokenCount; ++i) {

    145: for (uint256 i; i < poolLength; ++i) {

    208: for (uint256 i; i < vTokenCount; ++i) {

    229: for (uint256 i; i < rewardsDistributors.length; ++i) {

    263: for (uint256 i; i < markets.length; ++i) {

    391: for (uint256 i; i < vTokenCount; ++i) {

    418: for (uint256 i; i < markets.length; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

    File: Shortfall/Shortfall.sol

    175: for (uint256 i; i < marketsCount; ++i) {

    223: for (uint256 i; i < marketsCount; ++i) {

    374: for (uint256 i; i < marketsCount; ++i) {

    389: for (uint256 i; i < marketsCount; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

    File: Rewards/RewardsDistributor.sol

    209: for (uint256 i; i < numTokens; ++i) {

    282: for (uint256 i; i < vTokensCount; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

    File: Pool/PoolRegistry.sol

    356: for (uint256 i = 1; i <= _numberOfPools; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

    File: RiskFund/RiskFund.sol

    166: for (uint256 i; i < marketsCount; ++i) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }

    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }

    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-03] Use !=0 in place of >0 when comparing uint

Using > 0 costs more gas than != 0 when used on a uint in a require() statement.

There are 4 instances of this issue in 2 files:

    File: VToken.sol

    1369: require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: RiskFund/RiskFund.sol

    82: require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

    83: require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");

    139: require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5);
            c1.optimized(5);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 x) public returns(uint8){
            require(x>0,"x is greater than 0");
        }
    }
    
    contract Contract1 {
        function optimized(uint8 x) public returns(uint8){
            require(x!=0,"x is greater than 0");
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 58911                                     | 326             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 327             | 327 | 327    | 327 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 59111                                     | 327             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 327             | 327 | 327    | 327 | 1       |

## [G-04] Use assembly to check for address(0)

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

There are 41 instances of this issue in 8 files:

    File: Comptroller.sol

    128: require(poolRegistry_ != address(0), "invalid pool registry address");

    962: require(address(newOracle) != address(0), "invalid price oracle address");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: VToken.sol

    72: require(admin_ != address(0), "invalid admin address");

    134: require(spender != address(0), "invalid spender address");

    196: require(minter != address(0), "invalid minter address");

    626: require(spender != address(0), "invalid spender address");

    646: require(spender != address(0), "invalid spender address");

    1399: if (shortfall_ == address(0)) {

    1408: if (protocolShareReserve_ == address(0)) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: Shortfall/Shortfall.sol

    137: require(convertibleBaseAsset_ != address(0), "invalid base asset address");

    138: require(address(riskFund_) != address(0), "invalid risk fund address");

    166: ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||

    167: (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||

    169: ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
    
    170: (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),

    180: if (auction.highestBidder != address(0)) {

    189: if (auction.highestBidder != address(0)) {

    214: block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),

    349: require(_poolRegistry != address(0), "invalid address");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

    File: Pool/PoolRegistry.sol

    225: require(beaconAddress != address(0), "PoolRegistry: Invalid Comptroller beacon address.");

    226: require(priceOracle != address(0), "PoolRegistry: Invalid PriceOracle address.");    

    257: require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");

    258: require(input.asset != address(0), "PoolRegistry: Invalid asset address");

    259: require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");

    260: require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

    264: _vTokens[input.comptroller][input.asset] == address(0),

    396: require(venusPool.creator == address(0), "PoolRegistry: Pool already exists in the directory.");

    422: if (address(shortfall_) == address(0)) {

    431: if (protocolShareReserve_ == address(0)) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

    File: RiskFund/RiskFund.sol

    80: require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");

    81: require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");

    100: require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");

    111: require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");

    127: require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");

    157: require(poolRegistry != address(0), "Risk fund: Invalid pool registry.");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

    File: BaseJumpRateModelV2.sol

    72: require(address(accessControlManager_) != address(0), "invalid ACM address");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol

    File: RiskFund/ProtocolShareReserve.sol

    40: require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

    41:require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");

    54: require(_poolRegistry != address(0), "ProtocolShareReserve: Pool registry address invalid");

    71: require(asset != address(0), "ProtocolShareReserve: Asset address invalid");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol

    File: Proxy/UpgradeableBeacon.sol

    8: require(implementation_ != address(0), "Invalid implementation address");

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(0x0000000000000000000000000000000000000001);
            c1.optimized(0x0000000000000000000000000000000000000001);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(address addr) public pure{
             if(addr == address(0)){
                revert();
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(address addr) public pure{
             assembly {
               if iszero(addr) {
                   revert(0x00, 0x20)
               }
            }
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 41893                                     | 240             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 258             | 258 | 258    | 258 | 1       |
    
| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 37687                                     | 219             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 252             | 252 | 252    | 252 | 1       |

## [G-05] Use assembly to write address storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 15 instances of this issue in 6 files:

    File: Comptroller.sol
    
    130: poolRegistry = poolRegistry_;
    
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: VToken.sol

    1390: underlying = underlying_;

    1403: shortfall = shortfall_;

    1412: protocolShareReserve = protocolShareReserve_;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: Shortfall/Shortfall.sol

    145: convertibleBaseAsset = convertibleBaseAsset_;

    351: poolRegistry = _poolRegistry;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol

    File: Pool/PoolRegistry.sol

    435: protocolShareReserve = protocolShareReserve_;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

    File: RiskFund/RiskFund.sol

    88: pancakeSwapRouter = pancakeSwapRouter_;

    89: minAmountToConvert = minAmountToConvert_;

    90: convertibleBaseAsset = convertibleBaseAsset_;

    118: shortfall = shortfallContractAddress_;

    129: pancakeSwapRouter = pancakeSwapRouter_;

    141: minAmountToConvert = minAmountToConvert_;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

    File: RiskFund/ProtocolShareReserve.sol

    45: protocolIncome = _protocolIncome;

    46: riskFund = _riskFund;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-06] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 12 instances of this issue in 4 files:

    File: Comptroller.sol
        
    130: if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
        
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: VToken.sol

    837: if (redeemTokens == 0 && redeemAmount > 0) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: Lens/PoolLens.sol

    462: if (deltaBlocks > 0 && borrowSpeed > 0) {

    483: if (deltaBlocks > 0 && supplySpeed > 0) {

    506: if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0) {

    526: if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol

    File: Rewards/RewardsDistributor.sol

    261: if (deltaBlocks > 0 && rewardTokenSpeed > 0) {

    348: if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {

    386: if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {

    418: if (amount > 0 && amount <= rewardTokenRemaining) {

    435: if (deltaBlocks > 0 && supplySpeed > 0) {

    463: if (deltaBlocks > 0 && borrowSpeed > 0) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |

## [G-07] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

There are 8 instances of this issue in 5 files:

    File: Comptroller.sol

    154: function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory) {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: VToken.sol

    59: function initialize(
    60:    address underlying_,
    61:    ComptrollerInterface comptroller_,
    62:    InterestRateModel interestRateModel_,
    63:    uint256 initialExchangeRateMantissa_,
    64:    string memory name_,
    65:    string memory symbol_,
    66:    uint8 decimals_,
    67:    address admin_,
    68:    address accessControlManager_,
    69:    RiskManagementInit memory riskManagement,
    70:    uint256 reserveFactorMantissa_
    71:) external initializer {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    163: function distributeBorrowerRewardToken(
    164:     address vToken,
    165:     address borrower,
    166:     Exp memory marketBorrowIndex
    167: ) external onlyComptroller {    

    187: function updateRewardTokenBorrowIndex(address vToken, Exp memory marketBorrowIndex) external onlyComptroller {

    197: function setRewardTokenSpeeds(
    198:     VToken[] memory vTokens,
    200:     uint256[] memory supplySpeeds,
    201:     uint256[] memory borrowSpeeds
    202: ) external {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol

    File: Pool/PoolRegistry.sol

    255: function addMarket(AddMarketInput memory input) external {

    343: function updatePoolMetadata(address comptroller, VenusPoolMetaData memory _metadata) external {

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol

    File: Factories/VTokenProxyFactory.sol

    28: function deployVTokenProxy(VTokenArgs memory input) external returns (VToken) {    

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |
    
    
| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-08] Avoid using state variable in emit(130 gas)

Using a state variable in SetOwner emits wastes gas.

There are 4 instances of this issue in 4 files:

    File: Comptroller.sol

    709: emit NewCloseFactor(oldCloseFactorMantissa, closeFactorMantissa);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: BaseJumpRateModelV2.sol

    162: emit NewInterestParams(baseRatePerBlock, multiplierPerBlock, jumpMultiplierPerBlock, kink);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol

    File: WhitePaperInterestRateModel.sol

    40: emit NewInterestParams(baseRatePerBlock, multiplierPerBlock);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol

    File: MaxLoopsLimitHelper.sol

    31: emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5);
            c1.optimized(5);
        }
    }
    
    contract Contract0 {
        uint8 num;
        function not_optimized(uint8 x) public returns(uint8){
            num = x;
            return num;
        }
    }
    
    contract Contract1 {
        uint8 num;
        function optimized(uint8 x) public returns(uint8){
            num = x;
            return x;
        }
    }

#### Gas Report

| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 44893                                     | 255             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 22417           | 22417 | 22417  | 22417 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 44093                                     | 251             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 22405           | 22405 | 22405  | 22405 | 1       |

## [G-09] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtructions act the same way.

There are 8 instances of this issue in 4 files:

    File: VToken.sol

    630: newAllowance += addedValue;

    652: currentAllowance -= subtractedValue;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol

    File: RiskFund/ReserveHelpers.sol

    66: assetsReserves[asset] += balanceDifference;

    67: poolsAssetsReserves[comptroller][asset] += balanceDifference;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol

    File: RiskFund/RiskFund.sol

    249: assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;

    250: poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol

    File: RiskFund/ProtocolShareReserve.sol

    74: assetsReserves[asset] -= amount;

    75: poolsAssetsReserves[comptroller][asset] -= amount;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.addOptimized();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function addOptimized() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| addOptimized                              | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-10] Gas saving is achieved by removing the delete keyword

The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete” key word is unnecessary.

There are 2 instances of this issue in 2 files:

    File: Comptroller.sol

    209: delete marketToExit.accountMembership[msg.sender];

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol

    File: Shortfall/Shortfall.sol

    379: delete auction.markets;

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol 

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        struct details{
            string name;
            uint8 age;
        }
        function not_optimized() public returns(uint8){
            details memory det = details({name:"John",age:17});
            delete det.name;
            det.name = "Alis";
        }
    }

    contract Contract1 {
        struct details{
            string name;
            uint8 age;
        }
        function optimized() public returns(uint8){
            details memory det = details({name:"John",age:17});
            det.name = "Alias";
        }
    }

#### Gas Report 

| src/test/GasTest.t.sol:Contract0 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 47699                                     | 268             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 320             | 320 | 320    | 320 | 1       |


| src/test/GasTest.t.sol:Contract1 contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 47099                                     | 265             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 308             | 308 | 308    | 308 | 1       |