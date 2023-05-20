### `delegateAllocations` in `JBXBuybackDelegate.payParams` can be built in a single line
```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

// Return this delegate as the one to use, and do not mint from the terminal
162 : delegateAllocations = new JBPayDelegateAllocation[](1);
163 : delegateAllocations[0] = 
164 :             JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});
````

This can be replaced with the above two lines making it a static array which is more efficient
```solidity
162 : JBPayDelegateAllocation[1] memory delegateAllocations = [JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value})]
```


### replace `immutable` with `constant` 
State variables `projectToken`, `pool`, `jbxTerminal`, `weth`, `_projectTokenIsZero`  can be set to `constant` since their value can be fixed at the compile time directly assigning the values instead of assigning through constructor.