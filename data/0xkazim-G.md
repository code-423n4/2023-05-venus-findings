# Findings Summary

| ID     | Title                                                  | Severity |
| ------ | ------------------------------------------------------ | -------- |
| [G-01] | using seperate require check will save gas             | gas      |
| [G-02] | no need to emit the old address/informations           | gas      |

# [G-01] using seperate require check will save gas        

## Description

using of seperate require check will be more gas effiecient than using `||` and `&&` this will save some gas at deploying time 
## context

you can change all `||` and `&&` to seperate require, one example of this :

```solidity

file: contract/shortfall.shorFall.sol

 require( 
        //audit gas use specifiec require  
            (auction.auctionType == AuctionType.LARGE_POOL_DEBT &&
                ((auction.highestBidder != address(0) && bidBps > auction.highestBidBps) ||
                    (auction.highestBidder == address(0) && bidBps >= auction.startBidBps))) ||
                (auction.auctionType == AuctionType.LARGE_RISK_FUND &&
                    ((auction.highestBidder != address(0) && bidBps < auction.highestBidBps) ||
                        (auction.highestBidder == address(0) && bidBps <= auction.startBidBps))),
            "your bid is not the highest"
        );
```
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L164-L172


```solidity 

 require(
            block.number > auction.highestBidBlock + nextBidderBlockLimit && auction.highestBidder != address(0),
            "waiting for next bidder. cannot close auction"
        );

```
## Recommendations

change all these require checks to seperate require



# [G-01]no need to emit the old address/informations        

## Description

emitting old address and information will cost more gas and there is no need to emit the old information or addresses.

## context

you can set the emits in way that emits only the new addresses and information this will save lot of gases

```solidity

function setShortfallContractAddress(address shortfallContractAddress_) external onlyOwner {
        require(shortfallContractAddress_ != address(0), "Risk Fund: Shortfall contract address invalid");
        require(
            IShortfall(shortfallContractAddress_).convertibleBaseAsset() == convertibleBaseAsset,
            "Risk Fund: Base asset doesn't match"
        );

        address oldShortfallContractAddress = shortfall;
        shortfall = shortfallContractAddress_;
        //gas:emit this only for new one 
       - emit ShortfallContractUpdated(oldShortfallContractAddress, shortfallContractAddress_);
    }```

```solidity 

  function setPancakeSwapRouter(address pancakeSwapRouter_) external onlyOwner {
        require(pancakeSwapRouter_ != address(0), "Risk Fund: PancakeSwap address invalid");
        address oldPancakeSwapRouter = pancakeSwapRouter;
        pancakeSwapRouter = pancakeSwapRouter_;
        //gas:emit this only for new one 

        emit PancakeSwapRouterUpdated(oldPancakeSwapRouter, pancakeSwapRouter_);
    }
```

```solidity 
function setMinAmountToConvert(uint256 minAmountToConvert_) external {
        _checkAccessAllowed("setMinAmountToConvert(uint256)");
        require(minAmountToConvert_ > 0, "Risk Fund: Invalid min amount to convert");
        uint256 oldMinAmountToConvert = minAmountToConvert;
        minAmountToConvert = minAmountToConvert_;
        //gas:emit this only for new one 
        emit MinAmountToConvertUpdated(oldMinAmountToConvert, minAmountToConvert_);
    }

```
## Recommendations

change all these emits to emit only new informations.