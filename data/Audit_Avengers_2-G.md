1) Repeated Calls to _getBlockNumber(): You can store the result of _getBlockNumber() in a local variable if it's going to be used multiple times in the same transaction.

One case, of many:

_getBlockNumber() is called 3 times via addReserves(). Can be optimized to call only once.

https://github.com/code-423n4/2023-05-venus/blob/723001cf7bc0f37aba26fb385ec1a60135f24fe3/contracts/VToken.sol#L350-L359
https://github.com/code-423n4/2023-05-venus/blob/723001cf7bc0f37aba26fb385ec1a60135f24fe3/contracts/VToken.sol#L678
https://github.com/code-423n4/2023-05-venus/blob/723001cf7bc0f37aba26fb385ec1a60135f24fe3/contracts/VToken.sol#L1177

In the initial implementation of the `addReserves` function, the `_getBlockNumber` function was being called three times: once in `addReserves`, once in `accrueInterest`, and once in `_addReservesFresh`. Each call to `_getBlockNumber` incurs a gas cost. To optimize the gas usage, we modified the `addReserves` function to call `_getBlockNumber` only once, store the returned value in the `currentBlockNumber` variable, and then pass `currentBlockNumber` as an argument to `accrueInterest` and `_addReservesFresh`. This way, we avoid the additional gas costs of the repeated `_getBlockNumber` calls.

This change should result in a significant reduction in gas costs when calling the `addReserves` function, as the `_getBlockNumber` function is now only called once instead of three times. However, the exact amount of gas saved will depend on the specific gas cost of a call to `_getBlockNumber` in the context of the Ethereum network at the time of the transaction.

function addReserves(uint256 addAmount) external override nonReentrant {
		uint256 currentBlockNumber = _getBlockNumber();
		accrueInterest(currentBlockNumber);
		_addReservesFresh(addAmount, currentBlockNumber);
}

function accrueInterest(uint256 currentBlockNumber) public virtual override returns (uint256) {
/* Remember the initial block number */
		//uint256 currentBlockNumber = _getBlockNumber();
		uint256 accrualBlockNumberPrior = accrualBlockNumber;

/* Short-circuit accumulating 0 interest */
		if (accrualBlockNumberPrior == currentBlockNumber) {
    		return NO_ERROR;
}
        
function _addReservesFresh(uint256 addAmount, uint256 currentBlockNumber) internal returns (uint256) {
		// totalReserves + actualAddAmount
		uint256 totalReservesNew;
		uint256 actualAddAmount;

// We fail gracefully unless market's block number equals current block number
		//if (accrualBlockNumber != _getBlockNumber()) {
		if (accrualBlockNumber != currentBlockNumber) {
    		revert AddReservesFactorFreshCheck(actualAddAmount);
}
