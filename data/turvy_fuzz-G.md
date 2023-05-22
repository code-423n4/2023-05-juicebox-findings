### Storage immutable variable can be cache to stack rather than reading from storage
storage variable bool _projectTokenIsZero is queried multiple times in a single function
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L265
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L267
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L271

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L225
Recommendation
cache value of _projectTokenIsZero to stack to save gas when querying

### subtraction can be performed in an unchecked block to save gas
_reservedToken is gotten from  _amountReceived - _nonReservedToken and wasn't modified. Therefore the operation _amountReceived - _reservedToken can't underflow
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

### Remove unnecessary calculation and extra stack assignment
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

_reservedToken is calculated by _amountReceived - _nonReservedToken
but _nonReservedTokenInContract is also calculated by _amountReceived - _reservedToken however the value _amountReceived and _reservedToken is still the same and never got changed before that, which means _nonReservedTokenInContract is still same as _nonReservedToken. Therefore unnecessary operation and extra stack assignment
Recommendation:
remove unnecessary operation and use the _nonReservedToken instead