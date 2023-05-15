# Summary

## Gas Optimization

| No     | Issue   |   Instances |
| ------ | --------- | ---    |
| [G-01] | should use arguments instead of state variable in emit. | 2 | - |
| [G-02] | Use struct when dealing with different input arrays to enforce array length matching. | 3 | - |
| [G-03] | Failure to check the zero address in the constructor causes the contract to be deployed again. | 2 | - |
| [G-04] | Transfer ERC20 immediately to the user. | 1 | - |
| [G-05] | Transfer erc20 immediately to Shortfall. | 1 | - |
| [G-06] | Remove unnecessary code to save gas. | 4 | - |
| [G-07] | Use calldata Instead of Memory for Function Parameters. | 3 | - |
| [G-08] | Use Modifiers Instead of Functions To Save Gas. | 1 | - |
| [G-09] | != 0 costs less gas compared to > 0 for unsigned integers in require statements. | 7 | - |
| [G-10] | <x> += <y> costs more gas than <x> = <x> + <y> for state variables. | 3 | - |
| [G-11] | Do not calculate constants. | 1 | - |
| [G-12] | Use hardcode address instead address(this). | 13 | - |
| [G-13] | require() Should Be Used Instead Of assert(). | 1 | - |
| [G-14] | Using XOR (^) and OR bitwise equivalents. | 1 | - |
| [G-15] | Using a positive conditional flow to save a NOT opcode. | 13 | - |
| [G-16] | Make 3 event parameters indexed when possible. | 10 | - |
| [G-17] | Use double if statements instead of &&. | 11 | - |
| [G-18] | Gas saving is achieved by removing the delete keyword (~60k) | 3 | - |
| [G-19] | Using delete instead of setting struct 0 saves gas. | 3 | - |
| [G-20] | Can make the variable outside the loop to save gas. | 5 | - |
| [G-21] | use assembly to check for address(0). | 16 | - |
| [G-22] | Use constants instead of type(uintx).max. | 5 | - |
| [G-23] | Use send() to move ether, but don’t check for success. | 2 | - |
| [G-24] | Use bitmaps to save gas.| 3 | - |
| [G-25] | Make functions and interfaces payable. | 2 | - |


### [G-01] should use arguments instead of state variable in emit

```solidity
file:   MaxLoopsLimitHelper.sol
31      emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L31

```solidity
file:   WhitePaperInterestRateModel.sol
40      emit NewInterestParams(baseRatePerBlock, multiplierPerBlock);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L40

### [G-02] Use struct when dealing with different input arrays to enforce array length matching

When the length of all input arrays needs to be the same, use a struct to combine multiple input arrays so you don't have to manually validate their lengths. 

```solidity
file:
151      function swapPoolsAssets(
        address[] calldata markets,
        uint256[] calldata amountsOutMin,
        address[][] calldata paths
    ) external override returns (uint256)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L151-L155

## Recommendation Code

```solidity

struct  swap {
        address  markets;
        uint256  amountsOutMin;
        uint256  paths;
    }

function swapPoolsAssets(setSwap[] calldata Pollsassets) external {
    // no need for length check
}

```

```solidity
file:   RewardsDistributor.sol
197     function setRewardTokenSpeeds(
        VToken[] memory vTokens,
        uint256[] memory supplySpeeds,
        uint256[] memory borrowSpeeds
    ) external

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L197-L201

```solidity
file:     Comptroller.sol
836       function setMarketBorrowCaps(VToken[] calldata vTokens, uint256[] calldata newBorrowCaps) external {
        _checkAccessAllowed("setMarketBorrowCaps(address[],uint256[])");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L836-L837

### [G-03] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.

This will prevent the contract from being deployed with a zero address for the accessControlManager\_ parameter, which could potentially cause errors or vulnerabilities in the contract. By including the zero address check, we can ensure that the contract is deployed correctly and efficiently.

```solidity
file:   JumpRateModelV2.sol
12      constructor(
        uint256 baseRatePerYear,
        uint256 multiplierPerYear,
        uint256 jumpMultiplierPerYear,
        uint256 kink_,
        IAccessControlManagerV8 accessControlManager_
    )
        BaseJumpRateModelV2(baseRatePerYear, multiplierPerYear, jumpMultiplierPerYear, kink_, accessControlManager_)
    /* solhint-disable-next-line no-empty-blocks */
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol#L12-L19

```solidity
file:   ProtocolShareReserve.sol
28      constructor() {
        // Note that the contract is upgradeable. Use initialize() or reinitializers
        // to set the state variables.
        _disableInitializers();
    }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L28-L32

### [G-04] Transfer ERC20 immediately to the user

```solidity
file:        Shortfall.sol
158           function placeBid(address comptroller, uint256 bidBps) external nonReentrant {
        Auction storage auction = auctions[comptroller];

        require(_isStarted(auction), "no on-going auction");
        require(!_isStale(auction), "auction is stale, restart it");
        require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
        require(
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );

        uint256 marketsCount = auction.markets.length;
        for (uint256 i; i < marketsCount; ++i) {
            VToken vToken = VToken(address(auction.markets[i]));
            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
                if (auction.highestBidder != address(0)) {
                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
                        MAX_BPS);
                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
                }

                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
            } else {
                if (auction.highestBidder != address(0)) {
                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
                }

                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
            }
        }

        auction.highestBidder = msg.sender;
        auction.highestBidBps = bidBps;
        auction.highestBidBlock = block.number;

        emit BidPlaced(comptroller, auction.startBlock, bidBps, msg.sender);
    }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L158-L202

## Recommendation Code

use comptroller instead of msg.sender

```solidit
                    erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
            } else {
                if (auction.highestBidder != address(0)) {
                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
                }

                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);



                 erc20.safeTransferFrom(comptroller, address(this), currentBidAmount);
            } else {
                if (auction.highestBidder != address(0)) {
                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
                }

                erc20.safeTransferFrom(comptroller, address(this), auction.marketDebt[auction.markets[i]]);
```

### [G-05] Transfer erc20 immediately to Shortfall

When you call the give() function in the Address or NFTDriver. The erc20 token are first getting send to those contracts and afterwards to the DripsHub contract. It’s also possible to send the tokens directly to the dripsHub contract.

```solidity
file:       Shortfall.sol
158           function placeBid(address comptroller, uint256 bidBps) external nonReentrant {
        Auction storage auction = auctions[comptroller];

        require(_isStarted(auction), "no on-going auction");
        require(!_isStale(auction), "auction is stale, restart it");
        require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
        require(
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );

        uint256 marketsCount = auction.markets.length;
        for (uint256 i; i < marketsCount; ++i) {
            VToken vToken = VToken(address(auction.markets[i]));
            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
                if (auction.highestBidder != address(0)) {
                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
                        MAX_BPS);
                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
                }

                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
            } else {
                if (auction.highestBidder != address(0)) {
                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
                }

                erc20.safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
            }
        }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L158-L195

## Recommendation Code

```solidity
            function placeBid(address comptroller, uint256 bidBps) external nonReentrant {
        Auction storage auction = auctions[comptroller];

        require(_isStarted(auction), "no on-going auction");
        require(!_isStale(auction), "auction is stale, restart it");
        require(bidBps <= MAX_BPS, "basis points cannot be more than 10000");
        require(
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );

        uint256 marketsCount = auction.markets.length;
        for (uint256 i; i < marketsCount; ++i) {
            VToken vToken = VToken(address(auction.markets[i]));
            IERC20Upgradeable erc20 = IERC20Upgradeable(address(vToken.underlying()));

            if (auction.auctionType == AuctionType.LARGE_POOL_DEBT) {
                if (auction.highestBidder != address(0)) {
                    uint256 previousBidAmount = ((auction.marketDebt[auction.markets[i]] * auction.highestBidBps) /
                        MAX_BPS);
                    erc20.safeTransfer(auction.highestBidder, previousBidAmount);
                }

                uint256 currentBidAmount = ((auction.marketDebt[auction.markets[i]] * bidBps) / MAX_BPS);
                erc20.safeTransferFrom(msg.sender, address(Shortfall), currentBidAmount);
            } else {
                if (auction.highestBidder != address(0)) {
                    erc20.safeTransfer(auction.highestBidder, auction.marketDebt[auction.markets[i]]);
                }

                erc20.safeTransferFrom(msg.sender, address(Shortfall), auction.marketDebt[auction.markets[i]]);
            }
        }

```

### [G-06] Remove unnecessary code to save gas

Remove unnecessary functions: The safe224 and safe32 functions are not used in this contract and can be safely removed to reduce the contract size and gas cost.
and also you could remove the isInterestRateModel boolean constant, as it is not strictly necessary for the contract to function. in they contract InterestRateModel.sol

```solidity
file:    ExponentialNoError.sol
63       function safe224(uint256 n, string memory errorMessage) internal pure returns (uint224) {
        require(n < 2**224, errorMessage);
        return uint224(n);
    }
68      function safe32(uint256 n, string memory errorMessage) internal pure returns (uint32) {
        require(n < 2**32, errorMessage);
        return uint32(n);
    }

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L63-L66
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L68-L71

## Recommendation Code

```solidity
struct Exp {
        uint128 mantissa;
    }

    struct Double {
        uint128 mantissa;
    }

    uint128 internal constant expScale = 1e18;
    uint128 internal constant doubleScale = 1e36;
    uint128 internal constant halfExpScale = expScale / 2;
    uint64 internal constant mantissaOne = uint64(expScale);

    function truncate(Exp memory exp) internal pure returns (uint128) {
        return exp.mantissa / expScale;
    }

    function mul_ScalarTruncate(
        Exp memory a,
        uint256 scalar
    ) internal pure returns (uint128) {
        Exp memory product = mul_(a, scalar);
        return truncate(product);
    }

    function mul_ScalarTruncateAddUInt(
        Exp memory a,
        uint256 scalar,
        uint256 addend
    ) internal pure returns (uint128) {
        Exp memory product = mul_(a, scalar);
        return add_(truncate(product), addend);
    }

    function lessThanExp(
        Exp memory left,
        Exp memory right
    ) internal pure returns (bool) {
        return left.mantissa < right.mantissa;
    }

    function add_(
        Exp memory a,
        Exp memory b
    ) internal pure returns (Exp memory) {
        return Exp({mantissa: add_(a.mantissa, b.mantissa)});
    }

    function add_(
        Double memory a,
        Double memory b
    ) internal pure returns (Double memory) {
        return Double({mantissa: add_(a.mantissa, b.mantissa)});
    }

    function add_(uint128 a, uint128 b) internal pure returns (uint128) {
        return a + b;
    }

    function sub_(
        Exp memory a,
        Exp memory b
    ) internal pure returns (Exp memory) {
        return Exp({mantissa: sub_(a.mantissa, b.mantissa)});
    }

    function sub_(
        Double memory a,
        Double memory b
    ) internal pure returns (Double memory) {
        return Double({mantissa: sub_(a.mantissa, b.mantissa)});
    }

    function sub_(uint128 a, uint128 b) internal pure returns (uint128) {
        return a - b;
    }

    function mul_(
        Exp memory a,
        Exp memory b
    ) internal pure returns (Exp memory) {
        return Exp({mantissa: mul_(a.mantissa, b.mantissa) / expScale});
    }

    function mul_(Exp memory a, uint256 b) internal pure returns (Exp memory) {
        return Exp({mantissa: mul_(a.mantissa, b)});
    }

    function mul_(uint256 a, Exp memory b) internal pure returns (uint128) {
        return mul_(a, b.mantissa) / expScale;
    }

    function mul_(
        Double memory a,
        Double memory b
    ) internal pure returns (Double memory) {
        return Double({mantissa: mul_(a.mantissa, b.mantissa) / doubleScale});
    }

    function mul_(Double memory a, uint256 b) internal pure returns (Double memory) {
        return Double({mantissa: mul_(a.mantissa, b)});
    }

    function mul_(uint256 a, Double memory b) internal pure returns (uint128) {
        return mul_(a, b.mantissa) / doubleScale;
    }

    function mul_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a * b;
    }

    function div_(
        Exp memory a,
        Exp memory b
    ) internal pure returns (Exp memory) {
        return Exp({mantissa: div_(mul_(a.mantissa, expScale), b.mantissa)});
    }

    function div_(Exp memory a, uint256 b) internal pure returns (Exp memory) {
        return Exp({mantissa: div_(a.mantissa, b)});
    }

    function div_(uint256 a, Exp memory b) internal pure returns (uint128) {
        return div_(mul_(a, expScale), b.mantissa);
    }

    function div_(
        Double memory a,
        Double memory b
    ) internal pure returns (Double memory) {
        return
            Double({mantissa: div_(mul_(a.mantissa, doubleScale), b.mantissa)});
    }

    function div_(Double memory a, uint256 b) internal pure returns (Double memory) {
        return Double({mantissa: div_(a.mantissa, b)});
    }

    function div_(uint256 a, Double memory b) internal pure returns (uint128) {
        return div_(mul_(a, doubleScale), b.mantissa);
    }

    function div_(uint256 a, uint256 b) internal pure returns (uint256) {
        return a / b;
    }

    function fraction(
        uint256 a,
        uint256 b
    ) internal pure returns (Double memory) {
        return Double({mantissa: div_(mul_(a, doubleScale), b)});
    }

```

```solidity
file:   InterestRateModel.sol
10      bool public constant isInterestRateModel = true;

```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol#L10

```solidity
file:    ReserveHelpers.sol
26       uint256[48] private __gap;

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L26

### [G-07] Use calldata Instead of Memory for Function Parameters

In some cases, having function arguments in calldata instead of memory is more optimal. When arguments are read-only on external functions, the data location should be calldata.

```solidity
file:   Comptroller.sol
154     function enterMarkets(address[] memory vTokens) external override returns (uint256[] memory)

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L154

```solidity
file:   RewardsDistributor.sol
197     function setRewardTokenSpeeds(
        VToken[] memory vTokens,
        uint256[] memory supplySpeeds,
        uint256[] memory borrowSpeeds
    ) external
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L197-L201

```solidity
file:   RiskFund.sol
151     function swapPoolsAssets(
        address[] calldata markets,
        uint256[] calldata amountsOutMin,
        address[][] calldata paths
    ) external override returns (uint256)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L151-L155

### [G-08] Use Modifiers Instead of Functions To Save Gas

```solidity
file:   Comptroller.sol
916     function setPriceOracle(PriceOracle newOracle) external onlyOwner {
        require(address(newOracle) != address(0), "invalid price oracle address");

        PriceOracle oldOracle = oracle;
        oracle = newOracle;
        emit NewPriceOracle(oldOracle, newOracle);
    }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961-L967

### [G-09] != 0 costs less gas compared to > 0 for unsigned integers in require statements

```solidity
file:  RiskFund.sol
82     require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
83     require(loopsLimit_ > 0, "Risk Fund: Loops limit can not be zero");
139    require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L82
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L83
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L139

```solidity
file:    VToken.sol
1369     require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1369

```solidity
file:   Comptroller.sol
367     if (snapshot.shortfall > 0)
1262    if (snapshot.shortfall > 0)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L367
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1262

```solidity
file:  VToken.sol
818    if (redeemTokensIn > 0)
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L818

### [G-10] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

```solidity
file:   ReserveHelpers.so
66      assetsReserves[asset] += balanceDifference;
67      poolsAssetsReserves[comptroller][asset] += balanceDifference;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L66
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L67

```solidity
file:  VToken.sol
630    newAllowance += addedValue;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L630

### [G-11] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
file:     ExponentialNoError.sol
22        uint256 internal constant halfExpScale = expScale / 2;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol#L22

### [G-12] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity
file:    BaseJumpRateModelV2.sol
98       revert Unauthorized(msg.sender, address(this), signature);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L98

```soidity
file:   Comptroller.sol
501     if (seizerContract == address(this))
504     if (address(VToken(vTokenCollateral).comptroller()) != address(this))
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L501
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L504

```solidity
file:    PoolRegistry.sol
415      uint256 balanceBefore = token.balanceOf(address(this));
416      token.safeTransferFrom(from, address(this), amount);
417      uint256 balanceAfter = token.balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L415
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L416
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L417

```solidity
file:    ReserveHelpers.sol
59       uint256 currentBalance = IERC20Upgradeable(asset).balanceOf(address(this));

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L59

```solidity
file:    RewardsDistributor.sol
417      uint256 rewardTokenRemaining = rewardToken.balanceOf(address(this));
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L417

```solidit
file:    Shortfall.sol
187      erc20.safeTransferFrom(msg.sender, address(this), currentBidAmount);
193      .safeTransferFrom(msg.sender, address(this), auction.marketDebt[auction.markets[i]]);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L187
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L193

```solidity
file:  VToken.sol
527    uint256 balance = token.balanceOf(address(this));
749    comptroller.preMintHook(address(this), minter, mintAmount);
842     comptroller.preRedeemHook(address(this), redeemer, redeemTokens);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L527
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L749
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L842

### [G-13] require() Should Be Used Instead Of assert()

```solidity
file:    Comptroller.sol
225      assert(assetIndex < len);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L225

### [G-14] Using XOR (^) and OR (|) bitwise equivalents

```solidity
file:       Shortfall.sol
164             require(
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L164-L172




### [G-15] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas
Max savings according to yarn profile: 150 gas

The following function either revert or returns some value. To save some gas (NOT opcode costing 3 gas), switch to a positive statement:

```solidity
file:   BaseJumpRateModelV2.sol
97      if (!isAllowedToCall)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L97

```solidity
file:   Comptroller.sol
204     if (!marketToExit.accountMembership[msg.sender])
256     if (!markets[vToken].isListed)
333     if (!markets[vToken].isListed)
337     if (!markets[vToken].accountMembership[borrower])
395     if (!markets[vToken].isListed)
439     if (!markets[vTokenBorrowed].isListed)
442     if (!markets[vTokenCollateral].isListed)
510     if (!markets[seizerContract].isListed)
670     if (!markets[address(orders[i].vTokenBorrowed)].isListed)
673     if (!markets[address(orders[i].vTokenCollateral)].isListed)
1245    if (!markets[vToken].isListed)
1250    if (!markets[vToken].accountMembership[redeemer])
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L204
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L256
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L333
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L337
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L395
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L439
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L442
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L510
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L670
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L673
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1245
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1250

### [G-16] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:     Comptroller.sol

30        event MarketEntered(VToken vToken, address account);

    /// @notice Emitted when an account exits a market
    event MarketExited(VToken vToken, address account);

    /// @notice Emitted when close factor is changed by admin
    event NewCloseFactor(uint256 oldCloseFactorMantissa, uint256 newCloseFactorMantissa);

    /// @notice Emitted when a collateral factor is changed by admin
    event NewCollateralFactor(VToken vToken, uint256 oldCollateralFactorMantissa, uint256 newCollateralFactorMantissa);

    /// @notice Emitted when liquidation threshold is changed by admin
    event NewLiquidationThreshold(
        VToken vToken,
        uint256 oldLiquidationThresholdMantissa,
        uint256 newLiquidationThresholdMantissa
    );

    /// @notice Emitted when liquidation incentive is changed by admin
    event NewLiquidationIncentive(uint256 oldLiquidationIncentiveMantissa, uint256 newLiquidationIncentiveMantissa);

    /// @notice Emitted when price oracle is changed
    event NewPriceOracle(PriceOracle oldPriceOracle, PriceOracle newPriceOracle);

    /// @notice Emitted when an action is paused on a market
    event ActionPausedMarket(VToken vToken, Action action, bool pauseState);

    /// @notice Emitted when borrow cap for a vToken is changed
    event NewBorrowCap(VToken indexed vToken, uint256 newBorrowCap);

    /// @notice Emitted when the collateral threshold (in USD) for non-batch liquidations is changed
    event NewMinLiquidatableCollateral(uint256 oldMinLiquidatableCollateral, uint256 newMinLiquidatableCollateral);

    /// @notice Emitted when supply cap for a vToken is changed
    event NewSupplyCap(VToken indexed vToken, uint256 newSupplyCap);

    /// @notice Emitted when a rewards distributor is added
    event NewRewardsDistributor(address indexed rewardsDistributor);

    /// @notice Emitted when a market is supported
    event MarketSupported(VToken vToken);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L30-L70

```solidity
file:    MaxLoopsLimitHelper.sol
16       event MaxLoopsLimitUpdated(uint256 oldMaxLoopsLimit, uint256 newmaxLoopsLimit);

    /// @notice Thrown an error on maxLoopsLimit exceeds for any loop
    error MaxLoopsLimitExceeded(uint256 loopsLimit, uint256 requiredLoops);

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol#L16-L19

```solidity
file:  PoolRegistry.sol
113    event PoolRegistered(address indexed comptroller, VenusPool pool);

    /**
     * @dev Emitted when a pool name is set.
     */
    event PoolNameSet(address indexed comptroller, string oldName, string newName);

    /**
     * @dev Emitted when a pool metadata is updated.
     */
    event PoolMetadataUpdated(
        address indexed comptroller,
        VenusPoolMetaData oldMetadata,
        VenusPoolMetaData newMetadata
    );

    /**
     * @dev Emitted when a Market is added to the pool.
     */
    event MarketAdded(address indexed comptroller, address vTokenAddress);

    /**
     * @notice Event emitted when shortfall contract address is changed
     */
    event NewShortfallContract(address indexed oldShortfall, address indexed newShortfall);

    /**
     * @notice Event emitted when protocol share reserve contract address is changed
     */
    event NewProtocolShareReserve(address indexed oldProtocolShareReserve, address indexed newProtocolShareReserve);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L113-L142

```solidity
file:    ProtocolShareReserve.sol
22       event FundsReleased(address comptroller, address asset, uint256 amount);

    /// @notice Emitted when pool registry address is updated
    event PoolRegistryUpdated(address indexed oldPoolRegistry, address indexed newPoolRegistry);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L22-L25

```solidity
file:   ReserveHelpers.sol
30      event AssetsReservesUpdated(address indexed comptroller, address indexed asset, uint256 amount);

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L30

```solidity
file:    RewardsDistributor.sol
75           event RewardTokenSupplySpeedUpdated(VToken indexed vToken, uint256 newSpeed);

    /// @notice Emitted when a new borrow-side REWARD TOKEN speed is calculated for a market
    event RewardTokenBorrowSpeedUpdated(VToken indexed vToken, uint256 newSpeed);

    /// @notice Emitted when REWARD TOKEN is granted by admin
    event RewardTokenGranted(address recipient, uint256 amount);

    /// @notice Emitted when a new REWARD TOKEN speed is set for a contributor
    event ContributorRewardTokenSpeedUpdated(address indexed contributor, uint256 newSpeed);

    /// @notice Emitted when a market is initialized
    event MarketInitialized(address vToken);

    /// @notice Emitted when a reward token supply index is updated
    event RewardTokenSupplyIndexUpdated(address vToken);

    /// @notice Emitted when a reward token borrow index is updated
    event RewardTokenBorrowIndexUpdated(address vToken, Exp marketBorrowIndex);

    /// @notice Emitted when a reward for contributor is updated
    event ContributorRewardsUpdated(address contributor, uint256 rewardAccrued);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L75-L96

```solidity
file:    RiskFund.sol
38          event PoolRegistryUpdated(address indexed oldPoolRegistry, address indexed newPoolRegistry);

    /// @notice Emitted when shortfall contract address is updated
    event ShortfallContractUpdated(address indexed oldShortfallContract, address indexed newShortfallContract);

    /// @notice Emitted when PancakeSwap router contract address is updated
    event PancakeSwapRouterUpdated(address indexed oldPancakeSwapRouter, address indexed newPancakeSwapRouter);

    /// @notice Emitted when min amount out for PancakeSwap is updated
    event AmountOutMinUpdated(uint256 oldAmountOutMin, uint256 newAmountOutMin);

    /// @notice Emitted when minimum amount to convert is updated
    event MinAmountToConvertUpdated(uint256 oldMinAmountToConvert, uint256 newMinAmountToConvert);

    /// @notice Emitted when pools assets are swapped
    event SwappedPoolsAssets(address[] markets, uint256[] amountsOutMin, uint256 totalAmount);

    /// @notice Emitted when reserves are transferred for auction
    event TransferredReserveForAuction(address comptroller, uint256 amount);

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L38-L56

```solidity
file:    Shortfall.sol
100         event AuctionRestarted(address indexed comptroller, uint256 auctionStartBlock);

    /// @notice Emitted when pool registry address is updated
    event PoolRegistryUpdated(address indexed oldPoolRegistry, address indexed newPoolRegistry);

    /// @notice Emitted when minimum pool bad debt is updated
    event MinimumPoolBadDebtUpdated(uint256 oldMinimumPoolBadDebt, uint256 newMinimumPoolBadDebt);

    /// @notice Emitted when wait for first bidder block count is updated
    event WaitForFirstBidderUpdated(uint256 oldWaitForFirstBidder, uint256 newWaitForFirstBidder);

    /// @notice Emitted when next bidder block limit is updated
    event NextBidderBlockLimitUpdated(uint256 oldNextBidderBlockLimit, uint256 newNextBidderBlockLimit);

    /// @notice Emitted when incentiveBps is updated
    event IncentiveBpsUpdated(uint256 oldIncentiveBps, uint256 newIncentiveBps);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L100-L115

```solidity
file:   VTokenProxyFactory.sol
26      event VTokenProxyDeployed(VTokenArgs args);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol#L26

```solidity
file:  WhitePaperInterestRateModel.sol
29     event NewInterestParams(uint256 baseRatePerBlock, uint256 multiplierPerBlock);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol#L29

### [G-17] Use double if statements instead of &&

If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity
file:    Comptroller.sol
755      if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L755

```solidity
file:   PoolLens.sol
462     if (deltaBlocks > 0 && borrowSpeed > 0)
483     if (deltaBlocks > 0 && supplySpeed > 0)
506     if (borrowerIndex.mantissa == 0 && borrowIndex.mantissa > 0)
526     if (supplierIndex.mantissa == 0 && supplyIndex.mantissa > 0)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L462
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L483
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L506
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol#L526

```solidity
file:   RewardsDistributor.sol
261     if (deltaBlocks > 0 && rewardTokenSpeed > 0)
248     if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex)
386     if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex)
435     if (deltaBlocks > 0 && supplySpeed > 0)
463     if (deltaBlocks > 0 && borrowSpeed > 0)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L261
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L348
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L386
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L435
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L463

```solidity
file:   VToken.sol
842     if (redeemTokens == 0 && redeemAmount > 0)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L842

### [G-18] Gas saving is achieved by removing the delete keyword (~60k)

30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete” key word is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity
file:    Comptroller.sol
209      delete marketToExit.accountMembership[msg.sender];

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L209

```solidity
file:   RewardsDistributor.sol
224     delete lastContributorBlock[contributor];
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L224

```solidity
file:    Shortfall.sol
379     delete auction.markets;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L379

### [G-19] Using delete instead of setting struct 0 saves gas

```solidity
file:    ComptrollerStorage.sol
103      uint256 internal constant NO_ERROR = 0;

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L103

```solidity
file:  Shortfall.sol
370    auction.highestBidBps = 0;
371    auction.highestBidBlock = 0;
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L370
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L371

### [G-20] Can make the variable outside the loop to save gas

```solidity
file:     Comptroller.sol
690       for (uint256 i; i < marketsCount; ++i) {
            (, uint256 borrowBalance, ) = _safeGetAccountSnapshot(borrowMarkets[i], borrower);
932      for (uint256 i; i < rewardsDistributorsLength; ++i) {
            address rewardToken = address(rewardsDistributors[i].rewardToken());
1122       for (uint256 i; i < rewardsDistributorsLength; ++i) {
            address rewardToken = address(rewardsDistributors[i].rewardToken());
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L690-691
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L932-L933
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1122-L1123

```solidity
file:     PoolRegistry.sol
356       for (uint256 i = 1; i <= _numberOfPools; ++i) {
            address comptroller = _poolsByID[i];
            _pools[i - 1] = (_poolByComptroller[comptroller]);
        }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L356-L359

```solidity
file:   Shortfall.sol
389     for (uint256 i; i < marketsCount; ++i) {
            uint256 marketBadDebt = vTokens[i].badDebt();

```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L389-L390

### [G-21] use assembly to check for address(0)

```solidity
file:  BaseJumpRateModelV2.sol
72     require(address(accessControlManager_) != address(0), "invalid ACM address");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol#L72

```solidity
file:   Comptroller.sol
128     require(poolRegistry_ != address(0), "invalid pool registry address");
962     require(address(newOracle) != address(0), "invalid price oracle address");
1281    return _getHypotheticalLiquiditySnapshot(account, VToken(address(0)), 0, 0, weight);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L128
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L962
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L1281

```solidity
file:     PoolRegistry.sol
257             require(input.comptroller != address(0), "PoolRegistry: Invalid comptroller address");
        require(input.asset != address(0), "PoolRegistry: Invalid asset address");
        require(input.beaconAddress != address(0), "PoolRegistry: Invalid beacon address");
        require(input.vTokenReceiver != address(0), "PoolRegistry: Invalid vTokenReceiver address");

263      require(
            _vTokens[input.comptroller][input.asset] == address(0),
            "PoolRegistry: Market already added for asset comptroller combination"
        );
396      require(venusPool.creator == address(0), "PoolRegistry: Pool already exists
in the directory.");
422      if (address(shortfall_) == address(0)) {
            revert ZeroAddressNotAllowed();
        }
431       if (protocolShareReserve_ == address(0)) {
            revert ZeroAddressNotAllowed();
        }
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L257-L260
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L263-L266
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L396
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422-L424
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L431-L433

```solidity
file:      ProtocolShareReserve.sol
40              require(_protocolIncome != address(0), "ProtocolShareReserve: Protocol Income address invalid");
        require(_riskFund != address(0), "ProtocolShareReserve: Risk Fund address invalid");
71     require(asset != address(0), "ProtocolShareReserve: Asset address invalid");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L40-L41
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol#L71

```solidity
file:  ReserveHelpers.sol
40    require(asset != address(0), "ReserveHelpers: Asset address invalid");
52       require(asset != address(0), "ReserveHelpers: Asset address invalid");
        require(poolRegistry != address(0), "ReserveHelpers: Pool Registry address is not set");
        require(
            PoolRegistryInterface(poolRegistry).getVTokenForAsset(comptroller, asset) != address(0),
            "ReserveHelpers: The pool doesn't support the asset"
        );
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L40
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol#L52-L57

```solidity
file:
80              require(pancakeSwapRouter_ != address(0), "Risk Fund: Pancake swap address invalid");
        require(convertibleBaseAsset_ != address(0), "Risk Fund: Base asset address invalid");
100     require(_poolRegistry != address(0), "Risk Fund: Pool registry address invalid");
111     require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L80-L81

### [G-22] Use constants instead of type(uintx).max

```solidity
file:  Comptroller.sol
262    if (supplyCap != type(uint256).max)
351    if (borrowCap != type(uint256).max)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L262
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L351

```solidity
file:  VToken.sol
1055   if (repayAmount == type(uint256).max)
1314    startingAllowance = type(uint256).max;
1331     if (startingAllowance != type(uint256).max)
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1055
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1314
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L1331


### [G-23] Use send() to move ether, but don’t check for success

The difference between send and transfer is that transfer reverts if the transfer fails, but send returns false. However, you can just ignore the return value of send, and that will result in fewer op codes. Ignoring return values is a very bad practice, and it's a shame the compiler doesn't stop you from doing that.

```solidity
file: VTokenInterfaces.sol
311   function transfer(address dst, uint256 amount) external virtual returns (bool);
```

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L311


```solidity
file:
114     function transferFrom(
        address src,
        address dst,
        uint256 amount
    ) external override nonReentrant returns (bool)
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L114-L118


### [G-24]  Use bitmaps to save gas
By using a bitmap, we can potentially save gas when storing and manipulating boolean values, especially when dealing with a large number of boolean values

```solidity
file:   ComptrollerStorage.sol
41      mapping(address => bool) accountMembership;
95      mapping(address => mapping(Action => bool)) internal _actionPaused;
101     mapping(address => bool) internal rewardsDistributorExists;
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L41
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L95
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol#L101



### [G-25] Make functions and interfaces payable

Although this is not dangerous per se, payable functions can lead to unexpected state changes, so it’s safer to not mark a function as payable if the intent is not to receive ether. But in a gas optimization contest, this will save some op codes, because non-payable functions explicitly check msg.value and revert if it is non-zero, whereas payable functions simply skip this check.

```solidity
file:      IProtocolShareReserve.sol
4          interface IProtocolShareReserve {
           function updateAssetsState(address comptroller, address asset) external;
}
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IProtocolShareReserve.sol#L4-L5

```solidity
file:   IShortfall.sol
4       interface IShortfall {
    function convertibleBaseAsset() external returns (address);
}
```
https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/IShortfall.sol#L4-L6