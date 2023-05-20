# JuiceBox Gas Optimization Findings

## Summary
The protocol team has surpised the auditors with their commitment to the gas efficiency . 

I appreciate their dedication towards user experience .

But as i wanted to contribute to the vision too , 

i was able to get findings that does not save much of the gas 

but they are considerable in terms of the overall experience 

that the protocol wants the user to feel in terms of gas efficiency.

Let's go through each:


### Plain-Gas-Efficient If conditions 
 We can use plain if conditionals to check the Existence of some value in a variable than to compare 
 the zero value using Not operator which takes 3 gas units.
#### Description
 For example, instead of using    if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

``` solidity
        if (_reservedToken != 0) 
```

We can use something like :

```solidity
        if (_reservedToken) 
```

This will save 3 units of gas just for one condition.
So if we have multiple conditions like this ,
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


## Gas efficient check for Provided project token to see if it is zero
Instead of checking the _projectToken supplied argument if it is zero , we can have a plain comparison of the variable if it contains any value. 

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L127

### Description

Look at the following lines in the protocol

```solidity
        _projectTokenIsZero = address(_projectToken) < address(_weth);
```
This uses following gas unit

#### Gas consumed

address keyword :  2 gas 
< operator      :  3 gas
address keyword :  2 gas

Total = 7 gas


What I propose will cut out only small gas but it's worth it.


```solidity
        _projectTokenIsZero = address(_projectToken) ==0x0000000000000000000000000000000000000000
```
#### Gas Consumed

address keyword                 : 2 gas
Eq (on assembly level )( == )   : 3 gas

Total = 5 Gas

## Gas Saved

Saving 2 Units of gas.
