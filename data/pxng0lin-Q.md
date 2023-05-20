## Issues
Immutable address arguments provided to the constructor are missing zero address checks this could result in unexpected behavior when attempting to use the contract.

[Link to code](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL118C4-L118C4)

File:   juice-buyback/contracts/JBXBuybackDelegate.sol

```
124:    projectToken = _projectToken;
125:    pool = _pool;
126:    jbxTerminal = _jbxTerminal;
127:    ...
128:    weth = _weth;
```

## Recommendation
Implement a zero address check using the require function and the != (inequality) operator with address(0).

Example:
```
// zero address check immutable contracts.
require(_projectToken != address(0), "Invalid project token address");
require(_pool != address(0), "Invalid pool address");
require(_jbxTerminal != address(0), "Invalid terminal address");
require(_weth != address(0), "Invalid weth address");

// assign after successful zero address checks.
projectToken = _projectToken;
pool = _pool;
jbxTerminal = _jbxTerminal;
_projectTokenIsZero = address(_projectToken) < address(_weth); // no change as used for bool true/false
weth = _weth;
```