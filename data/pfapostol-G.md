##### Summary

Gas estimation is performed using existing tests (forge test --gas-report) and self-written [program](https://github.com/PFAhard/sol_gas) for snapshots calculation.

# Gas Optimizations
|       | Issue                                                                  | Instances | Estimated gas(deployments) | Estimated gas(method call) |
| :---: | :--------------------------------------------------------------------- | :-------: | :------------------------: | :------------------------: |
|   1   | Store external immutable variables in storage to reduce external calls |     1     |           20 507           |            564             |
|   2   | Cast data directly to uint256                                          |     1     |            600             |             35             |
|   3   | Uncheck arithmetic that can't fail                                     |     2     |           4 200            |             74             |
|   4   | Reduce useless variables and operations                                |     1     |           3 000            |             38             |
|       | Overall                                                                |     5     |           26 314           |            685             |


---

## 1. **Store external immutable variables in storage to reduce external calls**

**NOTE**:The directory is an immutable variable in the terminal contract, so we can store it in the constructor.

Deployment gas cost reduced by **20 507**

Minimum functions call gas cost reduced by **544**

Average functions call gas cost reduced by **564**

Maximum functions call gas cost reduced by **610**


```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..d25742b 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -89,6 +89,8 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;

+    IJBDirectory public immutable directory;
+
     /**
      * @notice The WETH contract
      */
@@ -124,6 +126,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         projectToken = _projectToken;
         pool = _pool;
         jbxTerminal = _jbxTerminal;
+        directory = _jbxTerminal.directory();
         _projectTokenIsZero = address(_projectToken) < address(_weth);
         weth = _weth;
     }
@@ -287,7 +290,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw

         // If there are reserved token, add them to the reserve
         if (_reservedToken != 0) {
-            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+            IJBController controller = IJBController(directory.controllerOf(_data.projectId));

             // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
             controller.burnTokensOf({
@@ -332,7 +335,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      * @param  _amount the amount of token out to mint
      */
     function _mint(JBDidPayData calldata _data, uint256 _amount) internal {
-        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+        IJBController controller = IJBController(directory.controllerOf(_data.projectId));

         // Mint to the beneficiary with the fc reserve rate
         controller.mintTokensOf({
```

--- 

## 2. **Cast data directly to uint256**

**NOTE**:date contains only one variable (there will be no revert if input is incorrect)

Deployment gas cost reduced by **600**

Minimum functions call gas cost reduced by **23**

Average functions call gas cost reduced by **35**

Maximum functions call gas cost reduced by **40**

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..ef9876d 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -217,15 +217,12 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         // Check if this is really a callback
         if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

-        // Unpack the data
-        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
-
         // Assign 0 and 1 accordingly
         uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
         uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

         // Revert if slippage is too high
-        if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
+        if (_amountReceived < uint256(bytes32(data))) revert JuiceBuyback_MaximumSlippage();

         // Wrap and transfer the weth to the pool
         weth.deposit{value: _amountToSend}();

```

--- 

## 3. **Uncheck arithmetic that can't fail**

**NOTE:** Underflow is not possible, since the right side is a percentage of the left.

Deployment gas cost reduced by **4 200**

Minimum functions call gas cost reduced by **116**

Average functions call gas cost reduced by **74**

Maximum functions call gas cost reduced by **116**

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..e563203 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -280,7 +280,10 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         );

         // The amount to add to the reserved token
-        uint256 _reservedToken = _amountReceived - _nonReservedToken;
+        uint256 _reservedToken;
+        unchecked {
+            _reservedToken= _amountReceived - _nonReservedToken;
+        }

         // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
@@ -309,7 +312,10 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             });

             // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
-            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
+            uint256 _nonReservedTokenInContract;
+            unchecked {
+                _nonReservedTokenInContract= _amountReceived - _reservedToken;
+            }

             if (_nonReservedTokenInContract != 0) {
                 controller.burnTokensOf({
```

---

## 4. **Reduce useless variables and operations**

**NOTE**: `_nonReservedToken` is unchanged during the lifetime of the function

Deployment gas cost reduced by **3 000**

Minimum functions call gas cost reduced by **68**

Average functions call gas cost reduced by **38**

Maximum functions call gas cost reduced by **68**

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..6795b3b 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -309,13 +309,11 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             });

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
                     _memo: "",
                     _preferClaimedTokens: false
                 });
```

---

## 00.  **Overall gas effect** 

Deployment gas cost reduced by **26 314**

Minimum functions call gas cost reduced by **694**

Average functions call gas cost increased by **685**

Maximum functions call gas cost reduced by **777**

```diff
diff --git "a/gas-report\\orig.md" "b/gas-report\\fixed.md"
index cb57c9c..2f27a83 100644
--- "a/gas-report\\orig.md"
+++ "b/gas-report\\fixed.md"
@@ -1,23 +1,23 @@
 Running 4 tests for contracts/test/DelegateUnit.t.sol:TestUnitJBXBuybackDelegate
-[PASS] testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: 189895)
-[PASS] testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 265670)
-[PASS] testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (runs: 256, μ: 190807, ~: 190552)
-[PASS] testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 12291)
-Test result: ok. 4 passed; 0 failed; finished in 171.36ms
+[PASS] testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: 189940)
+[PASS] testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: 264848)
+[PASS] testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (runs: 256, μ: 190869, ~: 190597)
+[PASS] testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 12246)
+Test result: ok. 4 passed; 0 failed; finished in 167.43ms
 
 Running 6 tests for contracts/test/DelegateE2E.t.sol:TestIntegrationJBXBuybackDelegate
-[PASS] test_mintIfPreferClaimedIsFalse() (gas: 995415)
-[PASS] test_mintIfSlippageTooHigh() (gas: 1082925)
-[PASS] test_mintIfWeightGreatherThanPrice(uint256) (runs: 256, μ: 958009, ~: 958102)
-[PASS] test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1113600, ~: 1114547)
-[PASS] test_swapMultiple() (gas: 1361806)
-[PASS] test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088084, ~: 1084099)
-Test result: ok. 6 passed; 0 failed; finished in 2.88s
+[PASS] test_mintIfPreferClaimedIsFalse() (gas: 994686)
+[PASS] test_mintIfSlippageTooHigh() (gas: 1082151)
+[PASS] test_mintIfWeightGreatherThanPrice(uint256) (runs: 256, μ: 958072, ~: 958147)
+[PASS] test_swapIfQuoteBetter(uint256) (runs: 256, μ: 1112903, ~: 1113612)
+[PASS] test_swapMultiple() (gas: 1359936)
+[PASS] test_swapRandomAmountIn(uint256) (runs: 256, μ: 1088465, ~: 1084001)
+Test result: ok. 6 passed; 0 failed; finished in 3.00s
 | contracts/JBXBuybackDelegate.sol:JBXBuybackDelegate contract |                       |       |        |        |         |
 | ------------------------------------------------------------ | --------------------- | ----- | ------ | ------ | ------- |
 | Deployment Cost                                              | Deployment Size       |       |        |        |         |
-|                                                              | 1260474               | 6573  |        |        |         |    |
+|                                                              | 1234160               | 6595  |        |        |         |    |
 | Function Name                                                | min                   | avg   | median | max    | # calls |
-|                                                              | didPay                | 51675 | 161926 | 148266 | 242106  | 7  |
-|                                                              | payParams             | 2075  | 8271   | 9981   | 12476   | 10 |
-|                                                              | uniswapV3SwapCallback | 867   | 20125  | 28794  | 30714   | 6  |
+| didPay                                                       | 50981           | 160447 | 147482 | 241322 | 7       |
+| payParams                                                    | 2120            | 8313   | 10017  | 12521  | 10      |
+| uniswapV3SwapCallback                                        | 822             | 20084  | 28756  | 30676  | 6       |
```