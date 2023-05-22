### Unnecessary calculation and extra stack assignment

_reservedToken is calculated by _amountReceived - _nonReservedToken
but _nonReservedTokenInContract is also calculated by _amountReceived - _reservedToken however the value of `_amountReceived` and `_reservedToken` is still the same and never got changed before that, which means `_nonReservedTokenInContract` will still be same as `_nonReservedToken`. Therefore unnecessary operation and extra stack assignment
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

**Recommendation:**
remove unnecessary code line and use _nonReservedToken instead