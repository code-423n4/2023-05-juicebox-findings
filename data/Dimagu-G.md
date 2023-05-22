# C4 Audit Juicebox JBXBuybackDelegate (May 22, 2023)

## Gas Optimizations Submission

#### [G-1] Line 267: Arithmetic that is done on the fly can be stored as a constant.

Instead of calculating `TickMath.MAX_SQRT_RATIO - 1` and `TickMath.MIN_SQRT_RATIO + 1` in [line 267](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL267) every time swap is conducted, we can define this values as a constant variables.
Ex. `uint160 private constant MAX_SQRT_RATIO_MINUS_ONE = 1461446703485210103287273052203988822378723970341;` & `uint160 private constant MIN_SQRT_RATIO_PLUS_ONE = 4295128740;`, and we can get rid of `import "@uniswap/v3-core/contracts/libraries/TickMath.sol";` library.

_Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 838 gas_

|        | Med    | Max    | Avg    | # calls |
| ------ | ------ | ------ | ------ | ------- |
| Before | 148266 | 242106 | 161891 | 7       |
| After  | 148184 | 242024 | 161053 | 7       |

```solidity
File: src/EthRouter.sol
22:     import "@uniswap/v3-core/contracts/libraries/TickMath.sol";
​
258:    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
259:        internal
260:        returns (uint256 _amountReceived)
261:    {
262:        // Pass the token and min amount to receive as extra data
263:        try pool.swap({
264:            recipient: address(this),
265:            zeroForOne: !_projectTokenIsZero,
266:            amountSpecified: int256(_data.amount.value),
267:            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
268:            data: abi.encode(_minimumReceivedFromSwap)
269:        }) returns (int256 amount0, int256 amount1) {
270:            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
271:            _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
272:        } catch {
273:            // implies _amountReceived = 0 -> will later mint when back in didPay
274:            return _amountReceived;
275:        }
276:
277:        // The amount to send to the beneficiary
278:        uint256 _nonReservedToken = PRBMath.mulDiv(
279:            _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
280:        );
281:
282:        // The amount to add to the reserved token
283:        uint256 _reservedToken = _amountReceived - _nonReservedToken;
284:
285:        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
286:        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
287:
288:        // If there are reserved token, add them to the reserve
289:        if (_reservedToken != 0) {
290:            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
291:
292:            // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
293:            controller.burnTokensOf({
294:                _holder: address(this),
295:                _projectId: _data.projectId,
296:                _tokenCount: _reservedToken,
297:                _memo: "",
298:                _preferClaimedTokens: true
299:            });
300:
301:            // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
302:            controller.mintTokensOf({
303:                _projectId: _data.projectId,
304:                _tokenCount: _amountReceived,
305:                _beneficiary: address(this),
306:                _memo: _data.memo,
307:                _preferClaimedTokens: false,
308:                _useReservedRate: true
309:            });
310:
311:            // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
312:            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
313:
314:            if (_nonReservedTokenInContract != 0) {
315:                controller.burnTokensOf({
316:                    _holder: address(this),
317:                    _projectId: _data.projectId,
318:                    _tokenCount: _nonReservedTokenInContract,
319:                    _memo: "",
320:                    _preferClaimedTokens: false
321:                });
322:            }
323:        }
324:
325:        emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
326:    }
```

​

```diff
@@ -19,7 +19,6 @@ import "@paulrberg/contracts/math/PRBMath.sol";
​
import "@uniswap/v3-core/contracts/interfaces/IUniswapV3Pool.sol";
import "@uniswap/v3-core/contracts/interfaces/callback/IUniswapV3SwapCallback.sol";
-import "@uniswap/v3-core/contracts/libraries/TickMath.sol";
​
import "./interfaces/external/IWETH9.sol";
​
@@ -67,6 +66,16 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
     */
    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
​
+    /**
+     * @notice Uniswap's TickMath.MAX_SQRT_RATIO - 1
+     */
+    uint160 private constant MAX_SQRT_RATIO_MINUS_ONE = 1461446703485210103287273052203988822378723970341;
+
+    /**
+     * @notice Uniswap's TickMath.MIN_SQRT_RATIO + 1
+     */
+    uint160 private constant MIN_SQRT_RATIO_PLUS_ONE = 4295128740;
+
    //*********************************************************************//
    // --------------------- public constant properties ------------------ //
    //*********************************************************************//
@@ -264,7 +273,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
            recipient: address(this),
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(_data.amount.value),
-            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
+            sqrtPriceLimitX96: _projectTokenIsZero ? MAX_SQRT_RATIO_MINUS_ONE : MIN_SQRT_RATIO_PLUS_ONE,
            data: abi.encode(_minimumReceivedFromSwap)
        }) returns (int256 amount0, int256 amount1) {
            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
```

#### [G-2] Use unchecked to save gas

We checked several lines of code where arithmetic is done that we proofed to be more gas efficient when using unchecked.

1. [Line 156 to 167](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL156-LL167) can be rewritten to save gas:

```solidity
    if (_slippage > SLIPPAGE_DENOMINATOR) {
      revert JuiceBuyback_InvalidSlippage();
    }
    uint256 qs = _quote * _slippage;
    unchecked {
      // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
      if (_tokenCount < _quote - (qs / SLIPPAGE_DENOMINATOR)) {
        // Pass the quote and reserve rate via a mutex
        mintedAmount = _tokenCount;
        reservedRate = _data.reservedRate;

        // Return this delegate as the one to use, and do not mint from the terminal
        delegateAllocations = new JBPayDelegateAllocation[](1);
        delegateAllocations[0] = JBPayDelegateAllocation({
          delegate: IJBPayDelegate(this),
          amount: _data.amount.value
        });

        return (0, _data.memo, delegateAllocations);
      }
    }
```

_Gas Savings for `JBXBuybackDelegate.payParams`, obtained via protocol's tests: Avg 84 gas_
| | Med | Max | Avg | # calls |
| ------ | ----- | ----- | ---- | ------- |
| Before | 11040 | 13800 | 9774 | 13 |
| After | 10968 | 13709 | 9690 | 13 |

2. [Line 197](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL197) can be rewritten to save gas:

```solidity
    if (_slippage > SLIPPAGE_DENOMINATOR) {
      revert JuiceBuyback_InvalidSlippage();
    }
    uint256 qs = _quote * _slippage;
    uint256 _minimumReceivedFromSwap;
    unchecked {
      _minimumReceivedFromSwap = _quote - (qs / SLIPPAGE_DENOMINATOR);
    }
```

_Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 78 gas_

|        | Med    | Max    | Avg    | # calls |
| ------ | ------ | ------ | ------ | ------- |
| Before | 140264 | 246524 | 144896 | 10      |
| After  | 140187 | 246448 | 144818 | 7       |

3. [Line 278 - 280](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL278-LL280) can be rewritten to save gas:

```solidity
    if (_reservedRate > JBConstants.MAX_RESERVED_RATE) revert JuiceBuyback_InvalidReserveRate();
    uint256 _nonReservedRate;
    unchecked {
       _nonReservedRate = JBConstants.MAX_RESERVED_RATE - _reservedRate;
    }

    // The amount to send to the beneficiary
    uint256 _nonReservedToken = PRBMath.mulDiv(
      _amountReceived,
      _nonReservedRate,
      JBConstants.MAX_RESERVED_RATE
    );
```

_Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 15 gas_

|        | Med    | Max    | Avg    | # calls |
| ------ | ------ | ------ | ------ | ------- |
| Before | 140264 | 246524 | 144896 | 10      |
| After  | 140242 | 246503 | 144881 | 10      |