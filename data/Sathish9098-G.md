# GAS OPTIMIZATION

##

## [G-1] Using private rather than public for constants/immutable, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L124-L126) of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table

```solidity
FILE: Breadcrumbs2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

79: IERC20 public immutable projectToken;
85: IUniswapV3Pool public immutable pool;
90: IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;
95: IWETH9 public immutable weth;

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L79

##

## [G-2] The result of jbxTerminal.directory() external call value should be cached with immutables

jbxTerminal.directory() is an external call and the result can be determined at deployment time (i.e., it doesn't depend on any variable or dynamic state), you can consider saving the result in an immutable variable. This can potentially save gas by avoiding redundant external calls during contract execution.

At least this will save 2100 gas. 

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

290: IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
335: IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL290C12-L290C109

##

## [G-3] The mintedAmount and reservedRate values should be checked before assign values to avoid assign same values to state variables  

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

## [G-4] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

### MAX_SQRT_RATIO is constant variable and stores the default value. There is no possibility to overflow/underflow for fixed values 

```solidity
FILE: https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juicebuyback/contracts/JBXBuybackDelegate.sol#L267

267: sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,

``` 
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L267

##

## [G-5] Use a more recent version of solidity

Latest solidity version is 0.8.19 

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

## [G-6] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

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

##

## [G-7] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

### _amountToSend should be checked before call transfer() function

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

232: weth.transfer(address(pool), _amountToSend);

```















