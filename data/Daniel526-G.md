## [G-01] [Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L241-L243)
The function **`claimRewardToken`** is a public function that can be called by anyone, which could lead to high gas costs due to the large number of loops that can be executed. A possible solution to this issue is to impose a limit on the number of markets that can be claimed at once to reduce gas costs.
## [G-02] [Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L257-L292)
One possible optimization is to reduce the number of state changes made during the execution of a function. This can reduce the amount of gas required to execute the function, which can be beneficial for both the user and the contract owner.

The `rewardTokenAccrued` state variable is updated twice in the `updateContributorRewards` function, once during the calculation of the new accrued amount and once again at the end of the function. Instead, the value could be stored in a temporary variable and updated only once at the end of the function. This would reduce the number of state changes and thus the gas cost of the function.

Here's an example of how the code could be optimized:
```solidity
function updateContributorRewards(address contributor) public {
    uint256 rewardTokenSpeed = rewardTokenContributorSpeeds[contributor];
    uint256 blockNumber = getBlockNumber();
    uint256 deltaBlocks = sub_(blockNumber, lastContributorBlock[contributor]);
    
    if (deltaBlocks > 0 && rewardTokenSpeed > 0) {
        uint256 newAccrued = mul_(deltaBlocks, rewardTokenSpeed);
        rewardTokenAccrued[contributor] = add_(rewardTokenAccrued[contributor], newAccrued);
        lastContributorBlock[contributor] = blockNumber;

        emit ContributorRewardsUpdated(contributor, rewardTokenAccrued[contributor]);
    }
}
```
As you can see, the only change is that the line rewardTokenAccrued[contributor] = contributorAccrued; has been removed, and instead newAccrued is directly added to rewardTokenAccrued[contributor] within the if statement. This way, we avoid updating the storage variable unnecessarily and reduce the overall gas cost of the function.
## [G-03] **Use Less Storage**: 
[Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L397)
- The list of markets
- The list of users in each mark
- The list of tokens in each market
- The list of user balances for each token in each market

You can reduce the amount of storage used by storing this data off-chain and only storing the necessary data on-chain. For example, you could store the list of markets and the list of users in each market in a database. When a user wants to enter or exit a market, you could fetch the necessary data from the database and store it on-chain for the duration of the transaction.
## [G-04] **Use less computation** 
[Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L397)
The code above uses a lot of computation for the following:

- Calculating the user's balance in each market
- Calculating the amount of tokens that the user can mint or redeem
- Calculating the amount of interest that the user owes

You can reduce the amount of computation used by using more efficient algorithms. For example, you could use a logarithmic algorithm to calculate the user's balance in each market instead of a linear algorithm.
## [G-05] **Use more efficient data structures.**
[Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L138-L397)
The code above uses a number of inefficient data structures, such as arrays. You can reduce the gas costs by using more efficient data structures, such as mappings.
## [G-06] **Use libraries.**

```solidity
function calculateUserBalance(address user, address market) public view returns (uint256) {
  // Get the user's balance in the market.
  uint256 balance = markets[market].userBalances[user];

  // Add the user's borrow balance in the market.
  balance += markets[market].userBorrowBalances[user];

  // Return the user's total balance in the market.
  return balance;
}
```
This code calculates the user's balance in a market. It does this by first getting the user's balance in the market and then adding the user's borrow balance in the market.
This code can be improved by using the following library:
```solidity
import "libraries/SafeMath.sol";
```
The SafeMath library provides a number of functions for performing mathematical operations safely. One of these functions is the `add` function. The `add` function takes two numbers as arguments and returns the sum of the two numbers.
The following code shows how to use the SafeMath library to improve the code above:
```solidity
function calculateUserBalance(address user, address market) public view returns (uint256) {
  // Get the user's balance in the market.
  uint256 balance = markets[market].userBalances[user];

  // Add the user's borrow balance in the market.
  balance = SafeMath.add(balance, markets[market].userBorrowBalances[user]);

  // Return the user's total balance in the market.
  return balance;
}
```
This code uses the SafeMath library to add the user's balance and borrow balance safely. This reduces the gas costs of the code because the SafeMath library uses optimized code for performing mathematical operations.


### **Similarly**
SafeMath library provides safe versions of the arithmetic operations that can prevent overflow errors. Overflow errors can cause the contract to fail, and they can also waste gas.
For example, the `enterMarkets` function currently uses the following code to add the user's balance to the market's balance:
```solidity
balances[vToken] += amount;
```
This code can potentially overflow if the user's balance is large enough. To prevent this, the following code can be used instead:
```solidity
SafeMath.add(balances[vToken], amount);
```
This code will use the SafeMath library to add the user's balance to the market's balance. This will prevent overflow errors, and it will also save gas.
In general, using the SafeMath library for arithmetic operations can be a good way to optimize gas costs in a contract. This is because the SafeMath library provides safe versions of the arithmetic operations that can prevent overflow errors. Overflow errors can cause the contract to fail, and they can also waste gas.

## [G-07] **Batching**
[Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154C1-L235)
The `enterMarkets` function loops through an array of vTokens and performs the following operations for each vToken:
1. Checks if the vToken is listed.
2. Checks if the user is allowed to enter the market.
3. Adds the vToken to the user's assets.
If the length of the array is large, then the gas cost of this function could become expensive. This is because each iteration of the loop will incur a gas cost.
One potential optimization would be to use a batched approach to add multiple vTokens to the account's assets in a single transaction. This can be done by using the following code snippet:

```solidity
function enterMarketsBatch(address[] memory vTokens) public {
  // Check if the user is allowed to enter any of the markets.
  for (uint256 i = 0; i < vTokens.length; i++) {
    if (!markets[vTokens[i]].isListed) {
      revert MarketNotListed(vTokens[i]);
    }
  }

  // Add all of the vTokens to the user's assets.
  for (uint256 i = 0; i < vTokens.length; i++) {
    _addToMarket(vTokens[i], msg.sender);
  }
}
```
This code snippet will first check if the user is allowed to enter any of the markets. If the user is allowed to enter all of the markets, then the code snippet will add all of the vTokens to the user's assets in a single transaction.
This optimization can reduce the gas cost of adding multiple vTokens to the account's assets.
**SIMILAR SCENARIO**
[Link](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L154C1-L235)
The `exitMarket` function also loops through an array of vTokens and performs a series of operations for each vToken. These operations are:
1. Checks if the vToken is listed.
2. Checks if the user is allowed to exit the market.
3. Removes the vToken from the user's assets.
If the length of the array is large, then the gas cost of this function could become expensive. This is because each iteration of the loop will incur a gas cost.
One potential optimization would be to use a batched approach to remove multiple vTokens from the user's assets in a single transaction. This can be done by using the following code snippet:
```solidity
function exitMarketsBatch(address[] memory vTokens) public {
  // Check if the user is allowed to exit any of the markets.
  for (uint256 i = 0; i < vTokens.length; i++) {
    if (!markets[vTokens[i]].isListed) {
      revert MarketNotListed(vTokens[i]);
    }
  }

  // Remove all of the vTokens from the user's assets.
  for (uint256 i = 0; i < vTokens.length; i++) {
    _removeFromMarket(vTokens[i], msg.sender);
  }
}
```
This code snippet will first check if the user is allowed to exit any of the markets. If the user is allowed to exit all of the markets, then the code snippet will remove all of the vTokens from the user's assets in a single transaction.
This optimization can reduce the gas cost of removing multiple vTokens from the user's assets.