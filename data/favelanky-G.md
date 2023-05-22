## [G-01] Do not reset the values

Global values `mintedAmount` and `reservedRate` are cleared after usage. However, it is impossible for them to not be overridden in the `payParams` function in another call.
Here is how functions are called: `payParams` -> `didPay` (only when delegateAllocations is not 0 size). Consequently, if `delegateAllocations` is not zero size then`mintedAmount` and `reservedRate` will be overridden in `payParams`, otherwise `didPay` will not be called.
You will save ~100 gas for each external call of the global variable. ~200 gas will be saved overall.

```diff
function didPay(JBDidPayData calldata _data) external payable override {
		// Access control as minting is authorized to this delegate
		if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

		// Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
		uint256 _tokenCount = mintedAmount;
		// @gas useless clearing
-       mintedAmount = 1;

		// Retrieve the fc reserved rate and reset the mutex
		uint256 _reservedRate = reservedRate;
		// @gas useless clearing
-       reservedRate = 1;

		// The minimum amount of token received if swapping
		(, , uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
		uint256 _minimumReceivedFromSwap = _quote - ((_quote * _slippage) / SLIPPAGE_DENOMINATOR);

		// Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
		if (_data.preferClaimedTokens) {
			// Try swapping
			uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

			// If swap failed, mint instead, with the original weight + add to balance the token in
			if (_amountReceived == 0) _mint(_data, _tokenCount);
		} else {
			_mint(_data, _tokenCount);
		}
	}
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183-L209


## [G-02] Raise the check above and refactor the check

First of all `if(...)` check can be raised up before calculating `_amountToSend` variable. It will save a bit of gas. Moreover, you can use the ternary operation inside the if block. So you will not need to pay gas for storing local variables used only once in the check.

```diff

function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
		// Check if this is really a callback
		if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

		// Unpack the data
		uint256 _minimumAmountReceived = abi.decode(data, (uint256));

		// Assign 0 and 1 accordingly
-               uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
+               if (uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta)) < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
		uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

		// Revert if slippage is too high
-               if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

		// Wrap and transfer the weth to the pool
		weth.deposit{value: _amountToSend}();
		weth.transfer(address(pool), _amountToSend);
	}

```

