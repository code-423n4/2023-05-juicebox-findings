## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Save gas by removing duplicate/refactoring code | 1 |
| [G&#x2011;02] | Remove unused imports(ownable.sol) | 1 |

### [G&#x2011;01]  Save gas removing duplicate/refactoring code
This code can be refactored and duplicate code can be removed. This will reduce number of bytes used which ultimately saves some gas.

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

200        if (_data.preferClaimedTokens) {
201            // Try swapping
202            uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
203
204            // If swap failed, mint instead, with the original weight + add to balance the token in
205            if (_amountReceived == 0) _mint(_data, _tokenCount);
206        } else {
207            _mint(_data, _tokenCount);
208        }
```
[Link to code](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L200-L207)

### Recommended Mitigation steps

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

-        if (_data.preferClaimedTokens) {
-            // Try swapping
-            uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
-
-            // If swap failed, mint instead, with the original weight + add to balance the token in
-            if (_amountReceived == 0) _mint(_data, _tokenCount);

+        if (_data.preferClaimedTokens && _swap(_data, _minimumReceivedFromSwap, _reservedRate) == 0){
+              _mint(_data, _tokenCount)
+        } else {
+            _mint(_data, _tokenCount);
+        }
```

### [G&#x2011;02]  Remove unused imports(ownable.sol)
The contract is made ownable but it does not use ownable import functions in contract. Removing the unncessary imports saves some gas.

There is 1 instance of this issue.

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {
```
[Link to code](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L39)
