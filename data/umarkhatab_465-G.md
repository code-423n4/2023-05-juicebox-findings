
# Gas efficient check for Provided project token to see if it is zero
Instead of checking the _projectToken supplied argument if it is zero using following lines
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L127

```solidity
        _projectTokenIsZero = address(_projectToken) < address(_weth);
```
This uses following gas unit

address keyword :  2 gas 
< operator      :  3 gas
address keyword :  2 gas

Total = 7 gas


What I propose will cut out only small gas but it's worth it.


```solidity
        _projectTokenIsZero = address(_projectToken) ==0x0000000000000000000000000000000000000000
```

## This will save 2 units of gas as it uses gas as following

address keyword                 : 2 gas
Eq (on assembly level )( == )   : 3 gas

Total = 5 Gas

Saving 2 Units of gas.


My Gas estimation can be checked for other issues related to Gas Optimizations [Here👉](https://github.com/umaresso/JuiceBox-Findings/blob/main/GasOptimization-Umar.md)
