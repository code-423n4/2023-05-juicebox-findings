# LOW FINDINGS

##

## [L-1] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas

```diff
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

- 232: weth.transfer(address(pool), _amountToSend);
+ 232: (bool success, bytes memory data) = address(weth).call(abi.encodeWithSignature("transfer(address,uint256)", address(pool), _amountToSend));
```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL232C9-L232C53

# NON CRITICAL 

## [NC-2] 












[M‑01]	Fee-on-transfer/rebasing tokens will have problems when swapping	1
[M‑02]	Unsafe use of transfer()/transferFrom() with IERC20	1
[M‑03]	Return values of transfer()/transferFrom() not checked	1
[L‑01]	Unsafe downcast	1
[L‑02]	Loss of precision	2
[L‑03]	Use Ownable2Step rather than Ownable	1
[L‑04]	Vulnerable versions of packages are being used	1
[N‑01]	Constants in comparisons should appear on the left side	4
[N‑02]	Consider disabling renounceOwnership()	1
[N‑03]	Non-external variable and function names should begin with an underscore	2
[N‑04]	Imports could be organized more systematically	1
[N‑05]	Adding a return statement when the function defines a named return variable, is redundant	1
[N‑06]	constants should be defined rather than using magic numbers	1
[N‑07]	Event is not properly indexed	2
[N‑08]	Import declarations should import specific identifiers, rather than the whole file	17
[N‑09]	override function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings	1
[N‑10]	Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)	1
[N‑11]	Lines are too long	4
[N‑12]	Non-library/interface files should use fixed compiler versions, not floating ones	1
[N‑13]	Typos	3
[N‑14]	NatSpec @param is missing	9
[N‑15]	NatSpec @return argument is missing	2
[N‑16]	Not using the named return variables anywhere in the function is confusing	2
[N‑17]	Function ordering does not follow the Solidity style guide	1
[N‑18]	Contract does not follow the Solidity style guide's suggested layout ordering	1
[N‑19]	Contracts should have full test coverage	1
[N‑20]	Large or complicated code bases should implement invariant tests	1