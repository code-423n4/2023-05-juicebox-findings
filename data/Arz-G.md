## Summary

6/8 optimizations were benchmarked using the protocol's tests, i.e. using the following config: solc version 0.8.19, optimizer on, 200 runs and forge test --gas-report


### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | mulDivFixedPoint() can be used instead of mulDiv(,,10 ** 18) | 1 |  150 |
| [G&#x2011;02] | The return array can be used instead of creating a new JBPayDelegateAllocation[](0) array | 1 |  67 |
| [G&#x2011;03] | An empty string literal can be used instead of new bytes(0) | 1 |  74 |
| [G&#x2011;04] | Using if/else is cheaper than ternary operators in uniswapV3SwapCallback() | 2 |  7 |
| [G&#x2011;05] | reservedToken should be set after the transfer is made so you can save gas if the transfer reverts | 1 |  53 in the revert case |
| [G&#x2011;06] | The data projectId should be cached | 2 |  13 |
| [G&#x2011;07] | abi.encodePacked() is cheaper than abi.encode() | 1 |  - |
| [G&#x2011;08] | Optimize names to save gas | * |  - |

Total: 9 instances over 8 issues 
Total gas saved: 364


## Gas Optimizations

### [G&#x2011;01]  mulDivFixedPoint() can be used instead of mulDiv(,,10 ** 18)

The PRBMath library provides a mulDivFixedPoint() function where the dominator is always 10 ** 18. Because the dominator is always 10 ** 18 there this makes this function cheaper than the mulDiv() function where you pass in the dominator. Because the mulDiv() function uses 10 ** 18 in payParams() it can be replaced with mulDivFixedPoint() to save gas


*Gas Savings for payParams() obtained via protocol's tests: Avg 150 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  2128  |  8321  | 10024 |  12529   |  10 |
| After  |  2075  |  8271  | 9981 |  12476  | 10 |


*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L150



```solidity
File: contracts\JBXBuybackDelegate.sol

150:   uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

150:   uint256 _tokenCount = PRBMath.mulDivFixedPoint(_data.amount.value, _data.weight);

```

### [G&#x2011;02]  The return array can be used instead of creating a new JBPayDelegateAllocation[](0) array

When you are minting in the payParams() function it creates and then returns an empty JBPayDelegateAllocation array. However, even if the length of the array you are creating is 0, it still costs gas. Using the empty return array delegateAllocations will cost less gas than creating a new one and then returning it


*Gas Savings for payParams() obtained via protocol's tests: Avg 67 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  2128  |  8321  | 10024 |  12529   |  10 |
| After  |  2019  |  8254  | 9981 |  12476  | 10 |



*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L170

```solidity
File: contracts\JBXBuybackDelegate.sol

170:  return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));

```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

170:  return (_data.weight, _data.memo, delegateAllocations);

```



### [G&#x2011;03] An empty string literal can be used instead of new bytes(0)

An empty string literal is cheaper than creating new bytes(0) and paying more for the initialization.


*Gas Savings for didPay() mint obtained via protocol's tests: Avg 74 gas*


|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  51675  |  161126  | 148266 |  242106   |  7 |
| After  |  51675  |  161052  | 148266 |  242106  | 7 |




*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L349

```solidity
File: contracts\JBXBuybackDelegate.sol

348:   jbxTerminal.addToBalanceOf{value: _data.amount.value}(
349:       _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
350:   );


```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

348:   jbxTerminal.addToBalanceOf{value: _data.amount.value}(
349:       _data.projectId, _data.amount.value, JBTokens.ETH, "", ""
350:   );

```


### [G&#x2011;04]  Using if/else is cheaper than ternary operators in uniswapV3SwapCallback()

Setting amountReceived and amountToSend using ternary operators will cost more than using if/else

*There is 1 instance of this issue:*
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L225

*Gas Savings for uniswapV3SwapCallback() obtained via protocol's tests: Avg 7 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  867  |  20125  | 28794 |  30714   |  6 |
| After  |  859  |  20118  | 28788  | 30708  | 6 |



```solidity
File: contracts\JBXBuybackDelegate.sol

224:   uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
225:   uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

uint256 _amountReceived;
uint256 _amountToSend;

if(_projectTokenIsZero) {
  _amountReceived = uint256(-(amount0Delta));
  _amountToSend = uint256(amount1Delta);
}
else {
  _amountReceived = uint256(-(amount1Delta));
  _amountToSend = uint256(amount0Delta);
}  

```

### [G&#x2011;05] reservedToken should be set after the transfer is made so you can save gas if the transfer reverts

In the swap function, the reserverToken is set before the transfer and is only used after the transfer. It would be better to put it after the transfer function because the transfer can easily fail and you would have to pay for operations that you used to set the reservedToken


*Gas Savings for didPay() in the revert case when the transfer fails: Avg 53 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  2948  |  109288  | 112680 |  205775   |  6 |
| After  |   2882  |  109235  | 112597  | 205775  | 6 |



*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L283-L286

```solidity
File: contracts\JBXBuybackDelegate.sol

283: uint256 _reservedToken = _amountReceived - _nonReservedToken;
284:
285: // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
286: if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);


```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
uint256 _reservedToken = _amountReceived - _nonReservedToken;

```

### [G&#x2011;06] The data projectId should be cached

The swap and mint functions both access the data.projectId multiple times. Calldata loading is cheap but if you have a struct here it would be better to cache the projectId and then use it


*Gas Savings for didPay() swap btained via protocol's tests: Avg 11 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  51675  |  161126  | 148266 |  242106   |  7 |
| After  |  51655  |  161115 | 148246  | 242086  |7 |

*Gas Savings for didPay() mint obtained via protocol's tests: Avg 2 gas*

|        |   MIN  |   AVG  |   MED  |  MAX | # CALLS|
| ------ | ------ | ------ | ------ | ---- | ------ |
| Before |  51675  |  161126  | 148266 |  242106   |  7 |
| After  |  51675  |  161124 | 148266  | 242106  |7 |


*There is 2 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L334


```solidity
File: contracts\JBXBuybackDelegate.sol

///@audit The projectId should be cached like this;
uint256 projectId = _data.projectId;

IJBController controller = IJBController(jbxTerminal.directory().controllerOf(projectId));

```

### [G&#x2011;07] abi.encodePacked() is cheaper than abi.encode()

abi.encodePacked is cheaper than abi.encode() because packed encoding uses less data and creates a smaller output

Read more [here](https://twitter.com/bytes032/status/1613456938104233984)


*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L268

```solidity
File: contracts\JBXBuybackDelegate.sol

268: data: abi.encode(_minimumReceivedFromSwap)

```

*Should be replaced with:*

```solidity
File: contracts\JBXBuybackDelegate.sol

abi.encodePacked(_minimumReceivedFromSwap)

```

### [G&#x2011;08] Optimize names to save gas

You can save gas by ordering contracts most called functions via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save gas if you order the functions correctly

See this [link](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) to see how it works

