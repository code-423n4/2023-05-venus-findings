**ExponentialNoError.sol**
- L149 - Before doing the division, you should check that b is not 0 to handle the exception and not just that it occurs.

- L12/16 - Two structs are created: Exp and Double, which only have one variable inside, this is strange since the idea of ​​stucts is to group multiple variables in one concept.


**BaseJumpRateModelV2.sol**
- L141/158 - Before doing the division by (cash + borrows - reserves) it should be checked that it is not 0 to handle the exception and not just that it occurs.


**ComptrollerInterface.sol**
- The contract is called ComptrollerInterface and inside there are two interfaces: ComptrollerInterface and ComptrollerViewInterface, therefore the correct thing would be for the file to be called ComptrollerInterfaces.sol


**RiskFund/ProtocolShareReserve.sol**
- L18/19 - Two constants are created: protocolSharePercentage and baseUnit in storage, but they are never used, therefore it is an unnecessary expense in the deploy.


**WhitePaperInterestRateModel.sol**
- L96 - Before doing the division by (cash + borrows - reserves) it should be checked that it is not 0 to handle the exception and not just that it occurs.


**VTokenInterfaces.sol**
- L11/133 - The file is called VTokenInterfaces, but inside it contains a contract and an abstract, therefore, this is somewhat confusing when it comes to understanding the code.
If you can't implement it with interfaces, you should change the file name to VTokenStorage, for example.


**RiskFund/RiskFund.sol**
- It is confusing that the name of the folder is RiskFund and that the file has the same name, the idea of ​​a folder is to group multiple contracts in the same idea and in this case that would not be fulfilled.


**Rewards/RewardsDistributor.sol**
- L420/422 - The _grantRewardToken() function has only two possible return values: 0 or amount. Therefore, it could be simplified by using bool.

 
**Shortfall/Shortfall.sol**
- L359 - The _startAuction() function has 75 lines of code and there are parts that have many operations together, therefore to improve the understanding of the code
It would be beneficial to use helper functions with names that define what is done in that block of code.
