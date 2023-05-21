# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following 
config: `solc version 0.8.19`, `optimizer on`, and `200 runs`. I have turn on the optimizer, and its using
solidity version `0.8.19` because it has and open pragma; `pragma solidity ^0.8.16;`

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:| 
| [G-01](#g01) | Remove `Ownable`, since its not used | 1 | 
| [G-02](#g02) | `jbxTerminal.directory()` staticcall return can be set as `immutable` | 2 |
| [G-03](#g03) | `didPay(...)` Can be simplified | 1 |
| [G-04](#g04) | Ternary evaluation of `_amountReceived` and `_amountToSend` can be optimized | 1 | 
| [G-05](#g05) | `""` is cheaper than `new bytes(0)` | 1 |
| [G-06](#g06) | `sqrtPriceLimitX96` value can be `immutable` | 1 |
| [G-07](#g07) | Use `assembly` on safe math operations | 1 |
| [G-08](#g08) | Consider activate `via-ir` for deploying | 1 |


---
<h3 id="g01">
  Remove `Ownable`, since its not used
</h3>

Note, this will also reduce deploy size.

`Overall gas change: -90 (-0.001%)`

**Recommendation:**

<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..94068b6 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -12,7 +12,6 @@ import "@jbx-protocol/juice-contracts-v3/contracts/libraries/JBTokens.sol";
 import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBDidPayData.sol";
 import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBPayParamsData.sol";
 
-import "@openzeppelin/contracts/access/Ownable.sol";
 import "@openzeppelin/contracts/interfaces/IERC20.sol";
 
 import "@paulrberg/contracts/math/PRBMath.sol";
@@ -36,7 +35,7 @@ import "./interfaces/external/IWETH9.sol";
  *         liquidity, this delegate needs to be redeployed.
  */
 
-contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {
+contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback {
     using JBFundingCycleMetadataResolver for JBFundingCycle;
 
     //*********************************************************************//
```

</details>

---
<h3 id="g02">
  `jbxTerminal.directory()` staticcall return can be set as `immutable`
</h3>

The `jbxTerminal` is immutable, and according to documentation `JBController3_0_1.directory()` is immutable, therefore, `jbxTerminal.directory()` can be setup in a immutable.


```
test_swapIfQuoteBetter(uint256) (gas: -752 (-0.067%)) 
test_mintIfSlippageTooHigh() (gas: -752 (-0.069%)) 
test_mintIfPreferClaimedIsFalse() (gas: -752 (-0.076%)) 
test_swapMultiple() (gas: -1504 (-0.110%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -686 (-0.258%)) 
Overall gas change: -4446 (-0.061%)
```


**Recommendation:**

<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..dab3ab3 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -112,6 +112,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     uint256 private reservedRate = 1;
 
+    /// @dev The directory of the terminal
+    IJBDirectory private immutable _TERMINAL_DIRECTORY;
+
     /**
      * @dev No other logic besides initializing the immutables
      */
@@ -126,6 +129,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         jbxTerminal = _jbxTerminal;
         _projectTokenIsZero = address(_projectToken) < address(_weth);
         weth = _weth;
+        _TERMINAL_DIRECTORY = jbxTerminal.directory();
     }
 
     //*********************************************************************//
@@ -287,7 +291,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
 
         // If there are reserved token, add them to the reserve
         if (_reservedToken != 0) {
-            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+            IJBController controller = IJBController(_TERMINAL_DIRECTORY.controllerOf(_data.projectId));
 
             // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
             controller.burnTokensOf({
@@ -332,7 +336,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      * @param  _amount the amount of token out to mint
      */
     function _mint(JBDidPayData calldata _data, uint256 _amount) internal {
-        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+        IJBController controller = IJBController(_TERMINAL_DIRECTORY.controllerOf(_data.projectId));
 
         // Mint to the beneficiary with the fc reserve rate
         controller.mintTokensOf({
```

</details>


---
<h3 id="g03">
  `didPay(...)` Can be simplified
</h3>

Consider the next refactor in the `diff` to reduce the `if` branch.

```
test_swapIfQuoteBetter(uint256) (gas: -11 (-0.001%)) 
test_swapRandomAmountIn(uint256) (gas: -11 (-0.001%)) 
test_mintIfSlippageTooHigh() (gas: -11 (-0.001%)) 
test_swapMultiple() (gas: -22 (-0.002%)) 
test_mintIfPreferClaimedIsFalse() (gas: 27 (0.003%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -11 (-0.004%)) 
Overall gas change: -39 (-0.001%)
```

**Recommendation:**

<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..122be21 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -196,16 +196,14 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
         uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
 
+        uint256 _amountReceived;
         // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
         if (_data.preferClaimedTokens) {
             // Try swapping
-            uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
-
-            // If swap failed, mint instead, with the original weight + add to balance the token in
-            if (_amountReceived == 0) _mint(_data, _tokenCount);
-        } else {
-            _mint(_data, _tokenCount);
+            _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
         }
+        // If swap failed, mint instead, with the original weight + add to balance the token in
+        if (_amountReceived == 0) _mint(_data, _tokenCount);
     }
 
     /**
```

</details>

---
<h3 id="g04">
  Ternary evaluation of `_amountReceived` and `_amountToSend` can be optimized
</h3>


```
test_swapIfQuoteBetter(uint256) (gas: -21 (-0.002%)) 
test_swapRandomAmountIn(uint256) (gas: -21 (-0.002%)) 
test_mintIfSlippageTooHigh() (gas: -21 (-0.002%)) 
test_swapMultiple() (gas: -42 (-0.003%)) 
testRevertIfSlippageIsTooMuchWhenSwapping() (gas: -21 (-0.171%)) 
Overall gas change: -126 (-0.002%)
```

**Recommendation:**

<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..3648c77 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -221,9 +221,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
 
         // Assign 0 and 1 accordingly
-        uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
-        uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
-
+        (uint256 _amountReceived, uint256 _amountToSend) = (_projectTokenIsZero)
+            ? (uint256(-amount0Delta), uint256(amount1Delta))
+            : (uint256(-amount1Delta), uint256(amount0Delta));
         // Revert if slippage is too high
         if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
```

</details>

---
<h3 id="g05">
  Expression `""` is cheaper than `new bytes(0)`
</h3>


```
test_mintIfSlippageTooHigh() (gas: -259 (-0.024%)) 
test_mintIfPreferClaimedIsFalse() (gas: -259 (-0.026%)) 
Overall gas change: -518 (-0.007%)
```

**Recommendation:**

<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..cf7f54d 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -346,7 +346,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
 
         // Send the eth back to the terminal balance
         jbxTerminal.addToBalanceOf{value: _data.amount.value}(
-            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
+            _data.projectId, _data.amount.value, JBTokens.ETH, "", ""
         );
 
         emit JBXBuybackDelegate_Mint(_data.projectId);
```

</details>

---
<h3 id="g06">
  `sqrtPriceLimitX96` value can be `immutable` 
</h3>

Since `_projectTokenIsZero`and `TickMath.MAX_SQRT_RATIO` are values that never change, we can assume that the next expression will never change;
`_projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1`

**Recommendation:**

```
test_swapIfQuoteBetter(uint256) (gas: -106 (-0.010%)) 
test_swapRandomAmountIn(uint256) (gas: -106 (-0.010%)) 
test_mintIfSlippageTooHigh() (gas: -106 (-0.010%)) 
test_swapMultiple() (gas: -212 (-0.016%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -106 (-0.040%)) 
Overall gas change: -636 (-0.009%)
```


<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..3c533a8 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -94,6 +94,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     IWETH9 public immutable weth;
 
+    /// @dev used in `_swap` for `pool.swap`
+    uint160 private immutable _SQRTPRICELIMITX96;
+
     //*********************************************************************//
     // --------------------- private stored properties ------------------- //
     //*********************************************************************//
@@ -126,6 +129,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         jbxTerminal = _jbxTerminal;
         _projectTokenIsZero = address(_projectToken) < address(_weth);
         weth = _weth;
+        _SQRTPRICELIMITX96 = _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1;
     }
 
     //*********************************************************************//
@@ -264,7 +268,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             recipient: address(this),
             zeroForOne: !_projectTokenIsZero,
             amountSpecified: int256(_data.amount.value),
-            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
+            sqrtPriceLimitX96: _SQRTPRICELIMITX96,
             data: abi.encode(_minimumReceivedFromSwap)
         }) returns (int256 amount0, int256 amount1) {
             // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
```

</details>

---
<h3 id="g07">
  Use `assembly` on safe math operations
</h3>

Instead of 
`uint256 value = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);` you could do;

```solidity
uint256 value;
assembly {
  value := mul(_quote, div(_slippage, SLIPPAGE_DENOMINATOR))
}
value = _quote - value;
```


```test_mintIfSlippageTooHigh() (gas: -519 (-0.048%)) 
test_mintIfPreferClaimedIsFalse() (gas: -515 (-0.052%)) 
test_mintIfWeightGreatherThanPrice(uint256) (gas: -860 (-0.090%)) 
test_swapIfQuoteBetter(uint256) (gas: -1065 (-0.096%)) 
testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: -1545 (-0.814%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -2231 (-0.840%)) 
testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (gas: -1994 (-1.046%)) 
test_swapMultiple() (gas: -20268 (-1.488%)) 
test_swapRandomAmountIn(uint256) (gas: -20337 (-1.876%)) 
testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 474 (3.856%)) 
Overall gas change: -48860 (-0.673%)
test_mintIfWeightGreatherThanPrice(uint256) (gas: -128 (-0.013%)) 
test_swapIfQuoteBetter(uint256) (gas: -269 (-0.024%)) 
test_swapRandomAmountIn(uint256) (gas: -269 (-0.025%)) 
test_mintIfSlippageTooHigh() (gas: -269 (-0.025%)) 
test_mintIfPreferClaimedIsFalse() (gas: -269 (-0.027%)) 
test_swapMultiple() (gas: -538 (-0.040%)) 
testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (gas: -128 (-0.067%)) 
testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: -128 (-0.067%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -269 (-0.101%)) 
Overall gas change: -2267 (-0.031%)
```


<details>
  <summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..9509faf 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -152,8 +152,14 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         // Unpack the quote from the pool, given by the frontend
         (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
 
+        uint256 _minimumReceivedFromSwap;
+        assembly {
+            _minimumReceivedFromSwap := mul(_quote, div(_slippage, SLIPPAGE_DENOMINATOR))
+        }
+        _minimumReceivedFromSwap = _quote - _minimumReceivedFromSwap;
+
         // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
-        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
+        if (_tokenCount < _minimumReceivedFromSwap) {
             // Pass the quote and reserve rate via a mutex
             mintedAmount = _tokenCount;
             reservedRate = _data.reservedRate;
@@ -194,7 +200,11 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
 
         // The minimum amount of token received if swapping
         (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
-        uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
+        uint256 _minimumReceivedFromSwap;
+        assembly {
+            _minimumReceivedFromSwap := mul(_quote, div(_slippage, SLIPPAGE_DENOMINATOR))
+        }
+        _minimumReceivedFromSwap = _quote - _minimumReceivedFromSwap;
 
         // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
         if (_data.preferClaimedTokens) {
```

</details>


---
<h3 id="g07">
  Consider activate `via-ir` for deploying
</h3>

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add `--via-ir` to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html


```
test_mintIfSlippageTooHigh() (gas: -519 (-0.048%)) 
test_mintIfPreferClaimedIsFalse() (gas: -515 (-0.052%)) 
test_mintIfWeightGreatherThanPrice(uint256) (gas: -860 (-0.090%)) 
test_swapIfQuoteBetter(uint256) (gas: -1065 (-0.096%)) 
testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() (gas: -1545 (-0.814%)) 
testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() (gas: -2231 (-0.840%)) 
testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) (gas: -1994 (-1.046%)) 
test_swapMultiple() (gas: -20268 (-1.488%)) 
test_swapRandomAmountIn(uint256) (gas: -20337 (-1.876%)) 
testRevertIfSlippageIsTooMuchWhenSwapping() (gas: 474 (3.856%)) 
Overall gas change: -48860 (-0.673%)
```

**Usage**

```bash
# Take a snapshot
forge snapshot --snap=baseSnap --via-ir
# Use diff a against 
forge snapshot --diff=baseSnap --via-ir
```