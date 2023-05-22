1.The redundant calculation results in the consumption of additional gas fees
JBXBuybackDelegate.sol#L312
the value of `_nonReservedTokenInContract` is equal `_nonReservedToken`

```solidity
-   uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

-   if (_nonReservedTokenInContract != 0) {
+   if (_nonReservedToken != 0) {
        controller.burnTokensOf({
            _holder: address(this),
            _projectId: _data.projectId,
-           _tokenCount: _nonReservedTokenInContract,
+           _tokenCount: _nonReservedToken,
            _memo: "",
            _preferClaimedTokens: false
        });
    }
```