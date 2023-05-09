








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