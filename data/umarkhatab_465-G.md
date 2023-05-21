# JuiceBox Gas Optimization Findings

# Summary
The protocol team has surpised the auditors with their commitment to the gas efficiency . 

I appreciate their dedication towards user experience .

But as i wanted to contribute to the vision too , 

i was able to get findings that does not save much of the gas 

but they are considerable in terms of the overall experience 

that the protocol wants the user to feel in terms of gas efficiency.

Let's go through each:


## [G-01] Plain-Gas-Efficient If conditions 
 We can use plain if conditionals to check the Existence of some value in a variable than to compare 
 the zero value using NOT(!) operator which takes 3 gas units.
### Description
 For example, instead of using    if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

``` solidity
        if (_reservedToken != 0) 
```

We can use something like :

```solidity
        if (_reservedToken) 
```

This will save 3 units of gas just for one condition.
So if we have multiple conditions like this,
which in our case are 2 located here

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L286

```solidity
 
 if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L289


``` solidity
 
 if (_reservedToken != 0) {
            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

            // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
            controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
                _tokenCount: _reservedToken,
                _memo: "",
                _preferClaimedTokens: true
            });
...
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L314


```solidity

            if (_nonReservedTokenInContract != 0) {
                controller.burnTokensOf({
                    _holder: address(this),
                    _projectId: _data.projectId,
                    _tokenCount: _nonReservedTokenInContract,
                    _memo: "",
                    _preferClaimedTokens: false
                });
...

```


#### Gas Saved

So even in one call , we will save <b> 3+3+3=9 Gas Units </b>

that are considerable in a highly gas efficient protcol like Juice Box .

As , thousands or Millions of people in the future would call this function indirectly ,

this can be a major thing to update.

Let's make the protocol super-gas-efficient in the history of blockchain 💪



<hr/>

## [G-02] Optimized naming of methods to reduce gas
We can optimize the method names using the information

from the report provided by Lead watson IIIIII.

You have the power to optimize your public/external function names

and public member variable names, which can lead to significant gas savings.

Take a look at this fascinating example on GitHub [Check here] (https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) .

By optimizing your contract interfaces and abstract contracts, 

you can ensure that the most frequently-called functions

consume the least amount of gas during method lookup. 

Did you know that method IDs with two leading zero bytes

can save an impressive 128 gas each during deployment?

Additionally, by strategically renaming functions to achieve lower method IDs,

you can save a remarkable 22 gas per call for each sorted position shifted.

Embrace this opportunity for gas optimization

Order functions based on their gas costs and frequency of use:

Place the most frequently called functions with lower gas costs first.

Consider the gas costs mentioned in the report to determine the order.

Avoid leading zeroes in function names to reduce gas costs.

We have the following instances that will be optimized with this approach:

```solidity

  function payParams(JBPayParamsData calldata _data)
        external
        override
        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
...

```

```solidity

   function didPay(JBDidPayData calldata _data) external payable override {
 

```

```solidity

function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
    

```

```solidity
 function redeemParams(JBRedeemParamsData calldata _data)
        external
        override
        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)



```

```solidity

    function supportsInterface(bytes4 _interfaceId) external pure override returns (bool) {



```

and other public variables.

# [G-03] Failure to verify the zero address during contract initialization results in potential redeployment

It has been observed that the contract examined in the audit lacks a verification step for zero addresses during deployment (calling the constructor). This omission leaves a window of opportunity for inadvertent deployment with a zero address, triggering the need for redeployment. Such redeployments consume additional gas and incur high deployment costs. It is crucial to address this vulnerability and ensure proper address validation during contract initialization to avoid unnecessary gas expenditure and potential deployment mishaps.

## [G-04] Use latest solidity compiler version for the smart contract 
Let's see what are the benefits of latest Solidity compiler versions until now

Compared to the current version of the contract that is v0.8.16:

### Solidity v0.8.17:

Security fix for a vulnerability that could have been used to exploit contracts

that used the callcode opcode.

Support for the memoryview type.

New compiler that is faster and more efficient than the previous compiler.

### Solidity v0.8.18:
Support for the sha3 and ripemd160 functions in the assembly keyword.

Improved error messages for pragma statements.

Fixed a bug that could have caused contracts to be incorrectly mined.

### Solidity v0.8.19:
Support for the function calldatasize() returns (uint256) function.

Improved error messages for contracts that use the callcode opcode.

Fixed a bug that could have caused contracts to be incorrectly mined.

### Solidity v0.8.20:

Support for the function calldatacopy(address dst, uint256 offset, uint256 length) function.

Improved error messages for contracts that use the calldatasize function.

Fixed a bug that could have caused contracts to be incorrectly mined.

These are just a few of the benefits of using newer Solidity compiler versions.

By using the latest version of Solidity, we can ensure that our smart contracts are 

secure, feature-rich, performant, and compatible 

with the latest versions of the Solidity compiler.

## [G-05] Use `bytes` instead of `string` to optimize gas 
There are many reasons on why we should use `bytes` instead of the strings.

One of the reasons are bytes variables are packed into storage, while strings are not. 

Packing means that the bytes are stored in a more efficient way, which can lead to lower gas costs.

So, it is generally advised to use bytes instead of strings in smart contracts.

This can help to optimize gas costs and improve the performance of your contract.


Total instances : 2

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L147

```solidity

        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L238

```solidity
        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
```


## [G-06] Add unchecked {} for subtractions where the operands cannot underflow because of a previous calculation
### Code picture

``` require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }```

### Description

In the following parts of the code, there is no point in checking for underflow and overflow 

because the previous calculations have ensured that.

```solidity

   196       (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
-> 197        uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);


```
we can save gas by writing 

```solidity
-> 197        uint256 _minimumReceivedFromSwap ;
unchecked{
_minimumReceivedFromSwap  = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
}

```
we can do the same for following scenarios :

```solidity
156 if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)
...
```
```solidity
283    uint256 _reservedToken = _amountReceived - _nonReservedToken;
...
```

```solidity
312   uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
...
```


<hr/>

This wraps up the report .

Thank you for reading.

Let's ROCK on Gas Optimizations  🎸😎
