# `_mint`: Cache `_data.project_id` in a memory variable

Add the following line at the top of the function and replace all usages of `_data.project_id` elsewhere in the function with the new memory variable.
```
uint256 projectId = _data.projectId;
```

This change saves 8 gas in `_mint`:

```
-TestIntegrationJBXBuybackDelegate:test_mintIfPreferClaimedIsFalse() (gas: 995415)
-TestIntegrationJBXBuybackDelegate:test_mintIfSlippageTooHigh() (gas: 1082925)
+TestIntegrationJBXBuybackDelegate:test_mintIfPreferClaimedIsFalse() (gas: 995407)
+TestIntegrationJBXBuybackDelegate:test_mintIfSlippageTooHigh() (gas: 1082917)
```

# Store terminal directory in an `immutable` variable

The directory that is stored in `jbxTerminal` is `immutable` in the `JBPayoutRedemptionPaymentTerminal3_1` contract and since `jbxTerminal` is also `immutable` in `JBXBuybackDelegate` that means that the directory will always be the same during `JBXBuybackDelegate`'s lifecycle. In turn this means that instead of always getting the directory via `jbxTerminal.directory()` (once in `_swap` and once in `_mint`), the directory can be fetched in the `constructor` once and stored as an `immutable` variable, then used elsewhere. This change increases the deployment cost of the contract but should decrease all runtime operations by a lot.

This change saves ~700-750 gas in both `_swap` and `_mint`:
```
-TestIntegrationJBXBuybackDelegate:test_mintIfPreferClaimedIsFalse() (gas: 995415)
-TestIntegrationJBXBuybackDelegate:test_mintIfSlippageTooHigh() (gas: 1082925)
-TestIntegrationJBXBuybackDelegate:test_mintIfWeightGreatherThanPrice(uint256) (runs: 256, μ: 958008, ~: 958102)
-TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113731, ~: 1114547)
-TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1361806)
-TestIntegrationJBXBuybackDelegate:test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088125, ~: 1084099)
-TestUnitJBXBuybackDelegate:testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: 189895)
-TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265670)
-TestUnitJBXBuybackDelegate:testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (runs: 256, μ: 190818, ~: 190552)
-TestUnitJBXBuybackDelegate:testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 12291)
+TestIntegrationJBXBuybackDelegate:test_mintIfPreferClaimedIsFalse() (gas: 994686)
+TestIntegrationJBXBuybackDelegate:test_mintIfSlippageTooHigh() (gas: 1082174)
+TestIntegrationJBXBuybackDelegate:test_mintIfWeightGreatherThanPrice(uint256) (runs: 256, μ: 958053, ~: 958147)
+TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1112980, ~: 1113796)
+TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1360304)
+TestIntegrationJBXBuybackDelegate:test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088126, ~: 1084100)
+TestUnitJBXBuybackDelegate:testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: 189940)
+TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265007)
+TestUnitJBXBuybackDelegate:testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (runs: 256, μ: 190863, ~: 190597)
+TestUnitJBXBuybackDelegate:testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 12269)
```

# `_swap`: `_reservedTokens` calculation cannot underflow

When calculating `_nonReservedToken` we know that it is at most equal to `_amountReceived` (when `_reservedRate` is set to zero) so the `_reservedToken` cannot possibly underflow, hence it can be done within an `unchecked` clause:
```
-        uint256 _reservedToken = _amountReceived - _nonReservedToken;
+        uint256 _reservedToken;
+        unchecked {
+            // We know from calculating _nonReservedToken above that it is less
+            // than or equal to _amountReceived, so this sub can't underflow.
+            _reservedToken = _amountReceived - _nonReservedToken;
+        }
```
This change is saving 74 gas in `_swap`:
```
-TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113731, ~: 1114547)
-TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1361806)
-TestIntegrationJBXBuybackDelegate:test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088125, ~: 1084099)
+TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113657, ~: 1114473)
+TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1361658)
+TestIntegrationJBXBuybackDelegate:test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088051, ~: 1084025)
-TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265670)
+TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265596)
```

# `_swap`: Avoid calculating `_nonReservedTokenInContract` in favor of `_nonReservedToken`

`_amountReceived` is comprised of `_reservedToken` and `_nonReservedToken` and `_nonReservedToken` is already derived once in `_swap` so there is no reason to repeat the calculation in order to burn the non-reserved tokens left in the delegate contract.
```
             // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
-            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
-
-            if (_nonReservedTokenInContract != 0) {
+            if (_nonReservedToken != 0) {
                 controller.burnTokensOf({
                     _holder: address(this),
                     _projectId: _data.projectId,
-                    _tokenCount: _nonReservedTokenInContract,
+                    _tokenCount: _nonReservedToken,
```
This change saves 85 gas in `_swap`:
```
-TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113731, ~: 1114547)
-TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1361806)
+TestIntegrationJBXBuybackDelegate:test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113646, ~: 1114462)
+TestIntegrationJBXBuybackDelegate:test_swapMultiple() (gas: 1361636)
-TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265670)
+TestUnitJBXBuybackDelegate:testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265585)
```