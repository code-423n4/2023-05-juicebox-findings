# GAS OPTIMIZATION


## [G-1] Save gas by checking against default jbxTerminal.directory()

jbxTerminal.directory() is an external call and the result can be determined at deployment time (i.e., it doesn't depend on any variable or dynamic state), you can consider saving the result in an immutable variable. This can potentially save gas by avoiding redundant external calls during contract execution.

At least this will save 2100 gas. 

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

290: IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
335: IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL290C12-L290C109

##

## [G-2] The mintedAmount and reservedRate values should be checked before assign values to avoid unwanted state variable write  

- it’ll save 2100 gas to not set it to that value again

- (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR) if this condition is true then only 
mintedAmount, reservedRate  values updated

- if particular condition false then values remains to 1 . 

### For every function call possible to save 4200 gas  

```diff
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

188: uint256 _tokenCount = mintedAmount;
+ 192:        uint256 _reservedRate = reservedRate;
+ if(mintedAmount! =1)
+ {
+ 189:        mintedAmount = 1;
+ }
+ if(reservedRate! =1)
+ {
+ 189:        reservedRate = 1;
+ }
        // Retrieve the fc reserved rate and reset the mutex
- 192:        uint256 _reservedRate = reservedRate;
-193:        reservedRate = 1;

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L188-L193

##

## [G-3] Use a more recent version of solidity

- Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

2: pragma solidity ^0.8.16;

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL2C1-L2C25

##

## [G-4] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

144: function payParams(JBPayParamsData calldata _data)
        external
        override
        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)

258: function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L144C52-L147













[G‑01]	internal functions only called once can be inlined to save gas	1	20
[G‑02]	Stack variable used as a cheaper cache for a state variable is only used once	1	3
[G‑03]	Constructors can be marked payable	1	21