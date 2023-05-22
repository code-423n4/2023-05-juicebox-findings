## Low Risk Issues


### [L&#x2011;01] Unwanted change in state variables
In `didPay()` - There are instances where `_mint()` and `_swap()` function fails under the hood and does not revert. In this method before the `_swap()` and `_mint()` two state variables(`mintedAmount` and `reservedRate`) are being updated which don't effect `_mint()` and `_swap()` internal functions. 
for unsuccessful tx value of both state variables should be same before the tx but in this case it would be reset to 1. which can create big confusion making this a low risk issue.
This Issue can be avoided by resetting them at the end of function after the `_swap()` and `_mint()` logic

*There are 2 instances of this issue:*

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
189:          mintedAmount = 1;
193:          reservedRate = 1;
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L189
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L193