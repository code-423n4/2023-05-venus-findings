# GAS OPTIMIZATION

##




[G-] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS 8


calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

[G-] The result of function calls should be cached rather than re-calling the function 3
L

The instances below point to the second+ call of the function within a single function

[G-] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.


[G-] use uint(1)/uint(2) instead of bool true/false 


[G-] Save gas by checking against default WETH address

external call la address check pannama check and value aa immutable ls store panni 2100 gas save pannalam 

You can save a Gcoldsload (2100 gas) in the address provider, plus the 100 gas overhead of the external call, for every receive(), by creating an immutable DEFAULT_WETH variable which will store the initial WETH address, and change the require statement to be: require(msg.ender == DEFAULT_WETH || msg.sender == <etc>).

[G-] Avoid emitting constants

A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas). The Stake and Withdraw events’ second indexed parameter is a constant for a majority of events emitted (with the exception of the events emitted in the _stakeLP() and _withdrawLP() functions) and is unecessary to emit since the value will never change. Alternatively, you can avoid incurring the Glogtopic (375 gas) per call to any function that emits Stake/Withdraw (with the exception of _stakeLP() and _withdrawLP()) by creating separate events for each staking/withdraw function and opt out of using the current indexed asset topic in each event. This way you can still query the different staking/withdraw events and will save 375 gas for each staking/withdraw function (with the exception of _stakeLP() and _withdrawLP()).

Note that the events emitted in the _stakeLP() and withdrawLP() functions are not considered for this issue since the second indexed parameter is for the LP storage variable, which can be changed via the configureLP() function.


[G-] <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables  

FOR EVERY CALL CAN SAVE 13 GAS


## [G-1] State variables should be cached in stack variables rather than re-reading them from storage

> Instances(2) 

> Approximate gas saved: 500 gas

Caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Less obvious fixes/optimizations include having local storage variables of mappings within state variable mappings or mappings within state variable structs, having local storage variables of structs within mappings, having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.


```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

minted state variable should be cached with stack variable 

141: if (newMinted < minted){ /// @audit minted cached
            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution); /// @audit minted cached
            minted = newMinted;
        }
        if (newCollateral < colbal){
            withdrawCollateral(msg.sender, colbal - newCollateral);
        }
        // Must be called after collateral withdrawal
        if (newMinted > minted){  /// @audit minted cached
            mint(msg.sender, newMinted - minted);  /// @audit minted cached
        }

241: if (amount > minted) revert RepaidTooMuch(amount - minted);
        minted -= amount;

349:  uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; 

```
[Position.sol#L141-L150](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L141-L150)

##

## [G-2] Using storage instead of memory for structs/arrays saves gas

> Instances(2]1) 

> Approximate gas saved: 2100 gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

159: Challenge memory copy = Challenge(

```
[MintingHub.sol#L159](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L159)


## [G-3] For events use 3 indexed rule to save gas 

> Instances(12)

Need to declare 3 indexed fields for event parameters. If the event parameter is less than 3 should declare all event parameters indexed 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

41: event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);
42: event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);
43: event PositionDenied(address indexed sender, string message); // emitted if closed by governance

```
[Position.sol#L41-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L41-L43)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

48: event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);
49: event ChallengeAverted(address indexed position, uint256 number);
50: event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);
51: event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
52: event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);

```
[MintingHub.sol#L48-L52](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L48-L52)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

90: event Delegation(address indexed from, address indexed to); // indicates a delegation
91: event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

```
[Equity.sol#L90-L91](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L90-L91)

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

52: event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
53: event MinterDenied(address indexed minter, string message);

```
[Frankencoin.sol#L52-L53](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L52-L53)

##

## [G-4] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

> Instances(6) 

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

45: mapping (address => uint256) public minters;
50: mapping (address => address) public positions;

```
[Frankencoin.sol#L45-L50](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L45-L50)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

83: mapping (address => address) public delegates;
88: mapping (address => uint64) private voteAnchor;

```
[Equity.sol#L83-L88](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L83-L88)

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

43: mapping (address => uint256) private _balances;
45: mapping (address => mapping (address => uint256)) private _allowances;

```
[ERC20.sol#L43-L45](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L43-L45)

##

## [G-5] Lack of input value checks cause a redeployment if any human/accidental errors

> Instances(24) 

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables.

If any human/accidental errors happen need to redeploy the contract so this create the huge gas lose 

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

_minApplicationPeriod value is not checked before assigning to MIN_APPLICATION_PERIOD 

constructor(uint256 _minApplicationPeriod) ERC20(18){
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }

```
[Frankencoin.sol#L59-L62](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59-L62)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
        setOwner(_owner);
        original = address(this);
        hub = _hub;
        price = _liqPrice;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
        mintingFeePPM = _mintingFeePPM;
        reserveContribution = _reservePPM;
        minimumCollateral = _minCollateral;
        challengePeriod = _challengePeriod;
        start = block.timestamp + initPeriod; // one week time to deny the position
        cooldown = start;
        expiration = start + _duration;
        limit = _initialLimit;
        
        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
    }

```
[Position.sol#L50-L70](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

constructor(address _zchf, address factory) {
        zchf = IFrankencoin(_zchf);
        POSITION_FACTORY = IPositionFactory(factory);
    }

```
[MintingHub.sol#L54-L57](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L54-L57)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

 constructor(Frankencoin zchf_) ERC20(18) {
        zchf = zchf_;
    }

```
[Equity.sol#L93-L95](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L93-L95)

```solidity
FILE: 2023-04-frankencoin/contracts/StablecoinBridge.sol

constructor(address other, address zchfAddress, uint256 limit_){
        chf = IERC20(other);
        zchf = IFrankencoin(zchfAddress);
        horizon = block.timestamp + 52 weeks;
        limit = limit_;
    }

```
[StablecoinBridge.sol#L26-L31](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L26-L31)

##

## [G-6] Use nested if and, avoid multiple check combinations

> Instances(4)

> Approximate Gas Saved: 36 gas 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas 

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

84:   if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:   if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
267:  if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

294:  if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

```
[Position.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L294)

##

## [G-7] No need to evaluate all expressions to know if one of them is true

> Instances(1) 

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

106: } else if (isMinter(spender) || isMinter(isPosition(spender))){
```
[Frankencoin.sol#L106](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L106)

##

## [G-8] The result of function calls should be cached rather than re-calling the function 

> Instances(2) 

In Solidity, caching repeated function calls can be an effective way to optimize gas usage, especially when the function is called frequently with the same arguments

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

totalSupply() value should be cached instead of calling multiple times 

84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)


```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

anchorTime() function results should be cached 

function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
        uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
        totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
        totalVotesAnchorTime = anchorTime();
    }
```
[Equity.sol#L144-L148](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144-L148)

##

## [G-9] Amounts should be checked for 0 before calling a transfer

> Instances(10) 

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

87:  _transfer(msg.sender, address(reserve), _applicationFee);
283: _transfer(address(reserve), msg.sender, _amount);
285: _transfer(address(reserve), msg.sender, reserveLeft);
254: _transfer(address(reserve), msg.sender, freedAmount - _amountExcludingReserve); 
```
[Frankencoin.sol#L87](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L87)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

279:    zchf.transfer(target, proceeds);
```
[Equity.sol#L279](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L279)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

108:  zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
129:  existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
225:  zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);

```
[MintingHub.sol#L108](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L108)
##

## [G-10] Don't declare the variable inside the loops 

> Instances(2) 

In every iterations the new variables instance created this will consumes more gas . So just declare variables outside the loop and only use inside to save gas 

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

192: for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];

312: for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];

``` 
[Equity.sol#L192-L193](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192-L193)

##

## [G-11] Empty blocks should be removed to save deployment cost 

> Instances(1) 

```solidity

FILE: ERC20.sol

240: function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {
    }

```
[ERC20.sol#L240](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol#L240)

##

## [G-12] Functions should be used instead of modifiers to save gas 

> Instances(3) 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

modifier noCooldown() {
        if (block.timestamp <= cooldown) revert Hot();
        _;
    }


    modifier noChallenge() {
        if (challengedAmount > 0) revert Challenged();
        _;
    }


    modifier onlyHub() {
        if (msg.sender != address(hub)) revert NotHub();
        _;
    }

```
[Position.sol#L373-L390](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L373-L390)

##

## [G-13] Avoid contract existence checks by using low level calls

> Instances(5) 

> Approximate gas saved: 500 gas

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

109:  return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();
243:  uint256 equity = zchf.equity();
279:  zchf.transfer(target, proceeds);
292:  uint256 capital = zchf.equity();
310:  require(zchf.equity() < MINIMUM_EQUITY); 

```
[Equity.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L109)

##

## [G-14] Sort Solidity operations using short-circuit mode

> Instances(3) 

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```

//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)

```

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

84:   if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:   if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
267:  if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)

##

## [G-15] Use assembly to check for address(0)

> Instances(3) 

Saves 6 gas per instance

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

152: require(recipient != address(0));
180: require(recipient != address(0));

```
[ERC20.sol#L152](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L152)

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

56: require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

```
[ERC20PermitLight.sol#L56](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L56)

##

## [G-16] Shorthand way to write if / else statement can reduce the deployment cost 

> Instances(7) 

The normal if / else statement can be refactored in a shorthand way to write it:

Increases readability
Shortens the overall SLOC

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

184: if (time >= exp){
            return 0;
        } else {
            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
        }

160: if (newPrice > price) {
            restrictMinting(3 days);
        } else {
            checkCollateral(collateralBalance(), newPrice);
        }

121: if (afterFees){
            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
        } else {
            return totalMint * (1000_000 - reserveContribution) / 1000_000;
        }

250: if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }

```
[Position.sol#L184-L188](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L184-L188)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

267: if (effectiveBid > fundsNeeded){
            zchf.transfer(owner, effectiveBid - fundsNeeded);
        } else if (effectiveBid < fundsNeeded){
            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
        }
```
[MintingHub.sol#L267-L271](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L267-L271)

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

141: if (balance <= minReserve){
        return 0;
      } else {
        return balance - minReserve;
      }

207: if (currentReserve < minterReserve()){
         // not enough reserves, owner has to take a loss
         return theoreticalReserve * currentReserve / minterReserve();
      } else {
         return theoreticalReserve;
      }


```
[Frankencoin.sol#L141-L145](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L141-L145)

### Recommended Mitigation 

```solidity

time >= exp ? return 0 : return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

##

## [G-17] The Less gas consuming condition checks should be on top 

When writing conditional statements in smart contracts, it is generally best practice to order the conditions so that the less gas-consuming checks are performed first. This can help to optimize the gas usage of the contract and improve its overall efficiency

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
86: if (minters[_minter] != 0) revert AlreadyRegistered();

```
[Frankencoin.sol#L84-L86](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L86)

##

## [G-18] Internal functions not called by contract can be removed to save gas

Removing unused functions can help to reduce the size of the compiled bytecode, which can reduce the gas costs of deploying the contract and executing its functions. It can also make the contract easier to read and maintain by removing unnecessary code

```solidity
FILE: 2023-04-frankencoin/contracts/MathUtil.sol

18: function _cubicRoot(uint256 _v) internal pure returns (uint256) {
39: function _power3(uint256 _x) internal pure returns(uint256) {

```
[MathUtil.sol#L18](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L18)

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

179: function _mint(address recipient, uint256 amount) internal virtual {
200: function _burn(address account, uint256 amount) internal virtual {

```
[ERC20.sol#L179](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L179)

##

## [G-19] Using bools for memory incurs overhead

Use uint256(1) and uint256(2) for true/false

```solidity


```








[G‑01]	Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate	3	-
[G‑02]	Structs can be packed into fewer storage slots	3	-
[G‑03]	Using storage instead of memory for structs/arrays saves gas	6	25200
[G‑04]	State variables should be cached in stack variables rather than re-reading them from storage	22	2134
[G‑05]	Multiple accesses of a mapping/array should use a local variable cache	13	546
[G‑06]	The result of function calls should be cached rather than re-calling the function	1	-
[G‑07]	internal functions only called once can be inlined to save gas	19	380
[G‑08]	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	3	255
[G‑09]	<array>.length should not be looked up in every loop of a for-loop	3	9
[G‑10]	++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops	37	2220
[G‑11]	require()/revert() strings longer than 32 bytes cost extra gas	61	-
[G‑12]	Optimize names to save gas	16	352
[G‑13]	Using bools for storage incurs overhead	3	51300
[G‑14]	>= costs less gas than >	1	3
[G‑15]	internal functions not called by the contract should be removed to save deployment gas	2	-
[G‑16]	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	1	5
[G‑17]	Splitting require() statements that use && saves gas	4	12
[G‑18]	Using private rather than public for constants, saves gas	8	-
[G‑19]	Division by two should use bit shifting	1	20
[G‑20]	Stack variable used as a cheaper cache for a state variable is only used once	10	30
[G‑21]	require() or revert() statements that check input arguments should be at the top of the function	12	-
[G‑22]	Use custom errors rather than revert()/require() strings to save gas	99	-
[G‑23]	Functions guaranteed to revert when called by normal users can be marked payable	22	462
[G‑24]	Constructors can be marked payable	11	231