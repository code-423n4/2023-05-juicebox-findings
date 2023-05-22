# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Unused parent contracts should be removed to save deployment cost| 108616 |  1 |
|G-02      |Make `mintedAmount` and `reservedRate` function parameters instead of storage variables| - |  2 |
|G-03      |Add unchecked block when the operands cannot over/underflow| 370 |  4 |
# Details
## [G-01] Unused parent contracts should be removed to save deployment cost
The JBXBuybackDelegate contract inherits the Openzeppelins Ownable contract. Since no function has the onlyOwner modifier, the contract is useless and should be removed from the imports and from inheritance. This save a lot of demployment cost.

|Before     | After|  Saved | 
|:----: | :---           |  :----:         |
|1,260,474      |1,151,858| 108,616 |

## [G-02] Make `mintedAmount` and `reservedRate` function parameters instead of storage variables
The two storage variables [`mintedAmount`](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L106) and [`reservedRate`](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L113) are used in the `payParams()` and `didPay()` function. Both of these functions are called by the terminal in one transaction. 

It will cost a total of 10000 gas in the `payParams()` function to change the values. SSTORE from non-zero to non-zero is 5000 if it's a cold slot. In the `didPay()` function it will only cost 100 gas for each because the storage slot is already written to. Since they are both reset to their default value in the same transaction, both of them get a gas refund of 2800 gas. Even with the gas refunds, this will still be 4600 gas. 

It will be much more gas efficiënt if you make them return parameters in the `payParams()` function and pass them to the terminal. So you can retrieve them in the `didPay()` function as function parameters. This will only involve some stack operations because the variables will all be in memory. This will also reduce the lines of code in both of the functions.

```diff
L144:
    function payParams(JBPayParamsData calldata _data)
        external
        override
-       returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
+       returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations, uint256 mintedAmount, uint256 reservedRate)
    {
        // Find the total number of tokens to mint, as a fixed point number with 18 decimals
        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

        // Unpack the quote from the pool, given by the frontend
        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

        // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
            // Pass the quote and reserve rate via a mutex
-           mintedAmount = _tokenCount;
-           reservedRate = _data.reservedRate;

            // Return this delegate as the one to use, and do not mint from the terminal
            delegateAllocations = new JBPayDelegateAllocation[](1);
            delegateAllocations[0] =
                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});
-           return (0, _data.memo, delegateAllocations);
+           return (0, _data.memo, delegateAllocations, _tokenCount, _date.reservedRate);
        }

        // If minting, do not use this as delegate
-       return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
+       return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0), 1, 1);
    }

L183:
-   function didPay(JBDidPayData calldata _data) external payable override {
+   function didPay(JBDidPayData calldata _data, uint256 mintedAmount, uint256 reservedRate) external payable override {
        // Access control as minting is authorized to this delegate
        if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

        // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
-       uint256 _tokenCount = mintedAmount;
-       mintedAmount = 1;

        // Retrieve the fc reserved rate and reset the mutex
-       uint256 _reservedRate = reservedRate;
-       reservedRate = 1;

        // The minimum amount of token received if swapping
        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
        uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

        // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
        if (_data.preferClaimedTokens) {
            // Try swapping
-           uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
+           uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, reservedRate);

            // If swap failed, mint instead, with the original weight + add to balance the token in
-           if (_amountReceived == 0) _mint(_data, _tokenCount);
+           if (_amountReceived == 0) _mint(_data, mintedAmount);
        } else {
-           _mint(_data, _tokenCount);
+           _mint(_data, mintedAmount);
        }
    }
```

## [G-03] Add unchecked block when the operands cannot over/underflow
Since solidity v0.8.0 there is an under the hood check for over/underflow. This involves some additional opcodes so it costs more gas. If you use an unchecked block, there are no checks for under/overflow so it's cheaper.

The only case where division can overflow is the expression type(int).min / (-1). Since max slippage is 10k there is no way that the slippage is bigger than the quote. That's why we can make this operation unchecked.

```diff
L156:
+   unchecked {
        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
            // Pass the quote and reserve rate via a mutex
            mintedAmount = _tokenCount;
            reservedRate = _data.reservedRate;

            // Return this delegate as the one to use, and do not mint from the terminal
            delegateAllocations = new JBPayDelegateAllocation[](1);
            delegateAllocations[0] =
                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});

            return (0, _data.memo, delegateAllocations);
        }
+   }
``` 
|Before     | After|  Saved | 
|:----: | :---           |  :----:         |
|8271      |8089| 182 |

Adding and subtracting 1 from a high constant has no risk of over/underflow
```diff
L263:
+   unchecked {
        try pool.swap({
            recipient: address(this),
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(_data.amount.value),
            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
            data: abi.encode(_minimumReceivedFromSwap)
        }) returns (int256 amount0, int256 amount1) {
            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
            _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
        } catch {
            // implies _amountReceived = 0 -> will later mint when back in didPay
            return _amountReceived;
        }
+   }
```
|Before     | After|  Saved | 
|:----: | :---     |  :----:    |
|161126      |161016| 110 |

An unchecked block can also be placed at other places but this requires to declare the variable outside of the unchecked block. So it will save less gas, around 30. This can be done for [`_reservedToken`](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L283), [`_nonReservedTokenInContract`](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312) and [`_minimumReceivedFromSwap`](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L197)
