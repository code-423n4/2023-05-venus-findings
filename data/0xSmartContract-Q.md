## Summary
### Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|L-01|JumpRateModelFactory  is suspicious of the reorg attack|  |
|L-02|Missing Event for  initialize| 8 |
|L-03|Avoid _shadowing_ `inherited state variables` | 1 |
|L-04|Vtoken.sol contract hasn’t `__Ownable2Step_init()`||
|N-05|Use a single file for all system-wide constants| 22 |

Total 5 issues

### L-01 JumpRateModelFactory  is suspicious of the reorg attack

The Factory contract allows users to create and initialize JumpRateModelV2. New JumpRateModelV2 contracts are created and deployed in a deterministic fashion:

**Context:**
```solidity

contracts/Factories/JumpRateModelFactory.sol:
   5  
   6: contract JumpRateModelFactory {
   7:     function deploy(
   8:         uint256 baseRatePerYear,
   9:         uint256 multiplierPerYear,
  10:         uint256 jumpMultiplierPerYear,
  11:         uint256 kink_,
  12:         IAccessControlManagerV8 accessControlManager_
  13:     ) external returns (JumpRateModelV2) {
  14:         JumpRateModelV2 rate = new JumpRateModelV2(
  15:             baseRatePerYear,
  16:             multiplierPerYear,
  17:             jumpMultiplierPerYear,
  18:             kink_,
  19:             accessControlManager_
  20:         );
  21: 
  22:         return rate;
  23:     }
  24: }


```

**Description:**
If users rely on the address derivation in advance any funds sent to the wallet could potentially be withdrawn by anyone else. All in all, it could lead to the theft of user funds.


**Recommendation:**
Deploy the  new contract via create2 with salt that includes msg.sender and  JumpRateModelFactory.sol address

### L-02 Missing Event for  initialize

**Context:**
```solidity
8 results - 7 files

contracts/Comptroller.sol:
  137       */
  138:     function initialize(uint256 loopLimit, address accessControlManager) external initializer {
  139          __Ownable2Step_init();

contracts/VToken.sol:
  58       */
  59:     function initialize(
  60          address underlying_,

contracts/Pool/PoolRegistry.sol:
  163       */
  164:     function initialize(
  165          VTokenProxyFactory vTokenFactory_,

contracts/Rewards/RewardsDistributor.sol:
  110       */
  111:     function initialize(
  112          Comptroller comptroller_,

  124  
  125:     function initializeMarket(address vToken) external onlyComptroller {
  126          uint32 blockNumber = safe32(getBlockNumber(), "block number exceeds 32 bits");

contracts/RiskFund/ProtocolShareReserve.sol:
  38       */
  39:     function initialize(address _protocolIncome, address _riskFund) external initializer {
  40          require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");

contracts/RiskFund/RiskFund.sol:
  72       */
  73:     function initialize(
  74          address pancakeSwapRouter_,

contracts/Shortfall/Shortfall.sol:
  130       */
  131:     function initialize(
  132          address convertibleBaseAsset_,



```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit


### L-03 Avoid _shadowing_ `inherited state variables`

**Context:**
```solidity
contracts/VToken.sol:
  147       */
  148:     function balanceOfUnderlying(address owner) external override returns (uint256) {
  149:         Exp memory exchangeRate = Exp({ mantissa: exchangeRateCurrent() });
  150:         return mul_ScalarTruncate(exchangeRate, accountTokens[owner]);  // @audit owner
  151:     }

node_modules/@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol:
  47       */
  48:     function owner() public view virtual returns (address) {
  49:         return _owner; // @audit owner
  50:     }


```
`totalSupply` is shadowed, 

The local variable name ` owner ` in the ` VToken.sol ` hase the same name and create shadowing

**Recommendation:**
Avoid using variables with the same name, including inherited in the same contract, if used, it must be specified in the NatSpec comments.

### L-03 Vtoken.sol contract hasn’t `__Ownable2Step_init()`

**Context:**
```diff
1 result - 1 file

contracts/VToken.sol:
  58       */
  59:     function initialize(
  60:         address underlying_,
  61:         ComptrollerInterface comptroller_,
  62:         InterestRateModel interestRateModel_,
  63:         uint256 initialExchangeRateMantissa_,
  64:         string memory name_,
  65:         string memory symbol_,
  66:         uint8 decimals_,
  67:         address admin_,
  68:         address accessControlManager_,
  69:         RiskManagementInit memory riskManagement,
  70:         uint256 reserveFactorMantissa_
  71:     ) external initializer {
  72:         require(admin_ != address(0), "invalid admin address");
+              __Ownable2Step_init();
  73: 
  74:         // Initialize the market
  75:         _initialize(
  76:             underlying_,
  77:             comptroller_,
  78:             interestRateModel_,
  79:             initialExchangeRateMantissa_,
  80:             name_,
  81:             symbol_,
  82:             decimals_,
  83:             admin_,
  84:             accessControlManager_,
  85:             riskManagement,
  86:             reserveFactorMantissa_
  87:         );
  88:     }


```

`Constructors` or `initializes` are replaced by internal initializer functions following the naming convention __{ContractName}_init. Since these are internal, you must always define your own public initializer function and call the parent initializer of the contract you extend.

### N-01 Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

```solidity
22 results - 10 files

contracts/BaseJumpRateModelV2.sol:
  13:     uint256 private constant BASE = 1e18;
  23:     uint256 public constant blocksPerYear = 10512000;
  24  

contracts/ComptrollerStorage.sol:
  103:     uint256 internal constant NO_ERROR = 0;
  106:     uint256 internal constant closeFactorMinMantissa = 0.05e18; // 0.05
  109:     uint256 internal constant closeFactorMaxMantissa = 0.9e18; // 0.9
  112:     uint256 internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9
  115:     bool internal constant _isComptroller = true;
  116  

contracts/ErrorReporter.sol:
  5:     uint256 public constant NO_ERROR = 0; // support legacy return codes

contracts/ExponentialNoError.sol:
  20:     uint256 internal constant expScale = 1e18;
  21:     uint256 internal constant doubleScale = 1e36;
  22:     uint256 internal constant halfExpScale = expScale / 2;
  23:     uint256 internal constant mantissaOne = expScale;
  24  

contracts/InterestRateModel.sol:
  10:     bool public constant isInterestRateModel = true;
  11  

contracts/VTokenInterfaces.sol:
   53:     uint256 internal constant borrowRateMaxMantissa = 0.0005e16;
   56:     uint256 internal constant reserveFactorMaxMantissa = 1e18;
   57  

  141       */
  142:     bool public constant isVToken = true;
  143  

contracts/WhitePaperInterestRateModel.sol:
  11  contract WhitePaperInterestRateModel is InterestRateModel {
  12:     uint256 private constant BASE = 1e18;
  13  

  16       */
  17:     uint256 public constant blocksPerYear = 2102400;
  18  

contracts/Rewards/RewardsDistributor.sol:
  28      /// @notice The initial REWARD TOKEN index for a market
  29:     uint224 public constant rewardTokenInitialIndex = 1e36;
  30  

contracts/RiskFund/ProtocolShareReserve.sol:
  18:     uint256 private constant protocolSharePercentage = 70;
  19:     uint256 private constant baseUnit = 100;
  20  

contracts/Shortfall/Shortfall.sol:
  59      /// @notice Max basis points i.e., 100%
  60:     uint256 private constant MAX_BPS = 10000;

```


