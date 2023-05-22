### [Gas-01] USING BOOLS FOR STORAGE INCURS OVERHEAD
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.
```solidity
    bool private immutable _projectTokenIsZero; // @audit G- Bool storage incurehead
```
*Instances(01)*

```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63
```

### [Gas-02] SOME `public` STATE VARIABLES COULD BE `private` THAT WILL CAUSE LESS GAS CONSUMPTION
*Instances(4)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L79
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L85
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L90
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L95
```

### [Gas-03] GAS SAVING CAN BE ACHIEVED BY CHANGING THE MODEL FOR ASSIGNING VALUE TO THE `stuct`
```solidity
// Pass the quote and reserve rate via a mutex
mintedAmount = _tokenCount; 
reservedRate = _data.reservedRate;

// Return this delegate as the one to use, and do not mint from the terminal
delegateAllocations = new JBPayDelegateAllocation[](1);
delegateAllocations[0] =
-                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});

+ delegateAllocations[0].delegate: IJBPayDelegate(this);
+ delegateAllocations[0].amount: _data.amount.value;
```
*Instances(1)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L164
```

### [Gas-04] `import` STATEMENTS COULD BE IMPROVED THAT HELP IN GAS OPTIMIZATION
Instead importing whole files, import specific required part
like:
```solidity
import {part1, part2} from 'PartContract.sol`;
```
*Instances(17)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L4-L24
```

