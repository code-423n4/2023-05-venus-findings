








[L‑01]	Division by zero not prevented	1
[L‑02]	Loss of precision	14
[L‑03]	require() should be used instead of assert()	1
[L‑04]	safeApprove() is deprecated	4
[L‑05]	Missing checks for address(0x0) when assigning values to address state variables	1
[L‑06]	Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions	8
[L‑07]	External calls in an un-bounded for-loop may result in a DOS	4

[N‑01]	Consider disabling renounceOwnership()	7
[N‑02]	Events are missing sender information	12
[N‑03]	Variables need not be initialized to zero	2
[N‑04]	Imports could be organized more systematically	4
[N‑05]	Large numeric literals should use underscores for readability	2
[N‑06]	Constants in comparisons should appear on the left side	45
[N‑07]	Variable names don't follow the Solidity style guide	17
[N‑08]	else-block not required	2
[N‑09]	Events may be emitted out of order due to reentrancy	5
[N‑10]	if-statement can be converted to a ternary	1
[N‑11]	Consider using named mappings	33
[N‑12]	Import declarations should import specific identifiers, rather than the whole file	96
[N‑13]	Adding a return statement when the function defines a named return variable, is redundant	2
[N‑14]	public functions not called by the contract should be declared external instead	2
[N‑15]	2**<n> - 1 should be re-written as type(uint<n>).max	2
[N‑16]	constants should be defined rather than using magic numbers	11
[N‑17]	Events that mark critical parameter changes should contain both the old and the new value	5
[N‑18]	Constant redefined elsewhere	1
[N‑19]	Inconsistent spacing in comments	3
[N‑20]	Lines are too long	22
[N‑21]	Typos	10
[N‑22]	File is missing NatSpec	10
[N‑23]	NatSpec @param is missing	47
[N‑24]	NatSpec @return argument is missing	27
[N‑25]	Event is not properly indexed	12
[N‑26]	Not using the named return variables anywhere in the function is confusing	14
[N‑27]	Duplicated require()/revert() checks should be refactored to a modifier or function	5
[N‑28]	Interfaces should be indicated with an I prefix in the contract name	3
[N‑29]	Numeric values having to do with time should use time units for readability	2
[N‑30]	Consider using delete rather than assigning zero/false to clear values	8
[N‑31]	Contracts should have full test coverage	1
[N‑32]	Large or complicated code bases should implement invariant tests	1
[N‑33]	internal functions not called by the contract should be removed	2