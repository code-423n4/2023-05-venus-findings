
# Low Risk and Non-Critical Issues

# [L-01] Use a modifier for access control (13 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L194
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L256
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L333
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L345
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L313
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L403
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L456
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L180
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L134
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L222
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L311
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L422
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L431



# [L-02] Add a timelock to critical functions (14 Instances)

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). 

Link to the code:

1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L927
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L961
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L973
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L505
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol#L515
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L348
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L181
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L249
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L188
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L198
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L99
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L110
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L126
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L205


# [N-01] According to the syntax rules, use  => mapping ( instead of  => mapping( using spaces as keyword (21 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L44
2.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol#L72
3.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L23
4.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L26
5.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L29
6.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L32
7.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L35
8.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L38
9.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L41
10.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L44
11.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L47
12.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol#L50
13.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L83
14.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L88
15.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L98
16.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L103
17.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol#L108
18.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol#L35
19.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L107
20.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L110
21.	https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol#L113
