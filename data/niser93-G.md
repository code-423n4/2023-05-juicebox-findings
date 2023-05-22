| GAS_N°| Title  | Test gas saving| Gas report avg saving|
|:--------:|:------:|:---------------:|:----:|
| [GAS-01](#gas-01---avoid-to-define-variable-which-are-used-oncea)  | Avoid to define variable which are used once      | 218               | 843|
| [GAS-02](#gas-02---using-alternative-formulation-in-order-to-avoid-to-call-variable-twice)  | Using alternative formulation in order to avoid to call variable twice      | 51               | 812|
| [GAS-03](#gas-03---avoid-to-define-useless-variable)  | Avoid to define useless variable      | 340              | 861
| [GAS-04](#gas-04---declaring-using-and-modifying-variable-only-where-it-needs)  | Declaring, using and modifying variable only where it needs      | 674        |98|
|||
|-| All optimizations applied|1240|206|


## Optimization Details
```
forge snapshot --diff
```
| Test                              | Name                                                          | Gas   |
|-----------------------------------|---------------------------------------------------------------|-------|
| TestIntegrationJBXBuybackDelegate | test_mintIfPreferClaimedIsFalse()                             | -618  |
| TestIntegrationJBXBuybackDelegate | test_mintIfSlippageTooHigh()                                  | -55   |
| TestIntegrationJBXBuybackDelegate | test_mintIfWeightGreatherThanPrice(uint256)                   | -3    |
| TestIntegrationJBXBuybackDelegate | test_swapIfQuoteBetter(uint256)                               | -129  |
| TestIntegrationJBXBuybackDelegate | test_swapMultiple()                                           | -258  |
| TestIntegrationJBXBuybackDelegate | test_swapRandomAmountIn(uint256)                              | -44   |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens()    | -3    |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateSwapIfPreferenceIsToClaimTokens()       | -116  |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) | -3    |
| TestUnitJBXBuybackDelegate        | testRevertIfSlippageIsTooMuchWhenSwapping()                   | -11   |
| --                                | --                                                            |       |
| Total                             |                                                               | -1240 |


```
forge test --gas-report
```

|    | Contract                          | Function name         | Gas avg   | Gas diff   |
|----|-----------------------------------|-----------------------|-----------|------------|
| -  | JBXBuybackDelegate                | didPay                | 161926    |            |
| +  | JBXBuybackDelegate                | didPay                | 161770    | -156       |
| -  | JBXBuybackDelegate                | payParams             | 8271      |            |
| +  | JBXBuybackDelegate                | payParams             | 8268      | -3         |
| -  | JBXBuybackDelegate                | uniswapV3SwapCallback | 20125     |            |
| +  | JBXBuybackDelegate                | uniswapV3SwapCallback | 20114     | -11        |
| -  | JBETHPaymentTerminal              | pay                   | 137702    |            |
| +  | JBETHPaymentTerminal              | pay                   | 137669    | -33        |
| -  | JBSingleTokenPaymentTerminalStore | recordPaymentFrom     | 44640     |            |
| +  | JBSingleTokenPaymentTerminalStore | recordPaymentFrom     | 44637     | -3         |
| -- | --                                | --                    |           |            |
|    | Total                             |                       |           | -206       |


## GAS-01 - Avoid to define variable which are used once

### Founded in [JBXBuybackDelegate.sol](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol)


[JBXBuybackDelegate.sol#L200-L208](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L200-L208)

```
if (_data.preferClaimedTokens) {
    // Try swapping
    uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

    // If swap failed, mint instead, with the original weight + add to balance the token in
    if (_amountReceived == 0) _mint(_data, _tokenCount);
} else {
    _mint(_data, _tokenCount);
}
```

[JBXBuybackDelegate.sol#L224-L228](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L228)

```
uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

// Revert if slippage is too high
if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
```

[JBXBuybackDelegate.sol#L311-L322](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L311-L322)

```
// 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

if (_nonReservedTokenInContract != 0) {
    controller.burnTokensOf({
        _holder: address(this),
        _projectId: _data.projectId,
        _tokenCount: _nonReservedTokenInContract,
        _memo: "",
        _preferClaimedTokens: false
    });
}
```

[JBXBuybackDelegate.sol#L334-L353](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L334-L353)

```
    function _mint(JBDidPayData calldata _data, uint256 _amount) internal {
        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

        // Mint to the beneficiary with the fc reserve rate
        controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amount,
            _beneficiary: _data.beneficiary,
            _memo: _data.memo,
            _preferClaimedTokens: _data.preferClaimedTokens,
            _useReservedRate: true
        });

        // Send the eth back to the terminal balance
        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );

        emit JBXBuybackDelegate_Mint(_data.projectId);
    }
```

#### Description
Avoid to declare variable if you use it once.
Sometimes it could get worse in human readability, so we report only cases where it is reasonable.

[JBXBuybackDelegate.sol#L200-L208](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L200-L208)

```diff
if (_data.preferClaimedTokens) {
    // Try swapping
-   uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

    // If swap failed, mint instead, with the original weight + add to balance the token in
-   if (_amountReceived == 0) _mint(_data, _tokenCount);
+   if (_swap(_data, _minimumReceivedFromSwap, _reservedRate) == 0) _mint(_data, _tokenCount);
} else {
    _mint(_data, _tokenCount);
}
```

[JBXBuybackDelegate.sol#L224-L228](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L228)

```diff
-   uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
    uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

    // Revert if slippage is too high
-   if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
+   if (uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta)) < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
```

[JBXBuybackDelegate.sol#L311-L322](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L311-L322)

```diff
    // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
-   uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

-   if (_nonReservedTokenInContract != 0) {
+   if (_amountReceived != _reservedToken) {
        controller.burnTokensOf({
            _holder: address(this),
            _projectId: _data.projectId,
-           _tokenCount: _nonReservedTokenInContract,
+           _tokenCount: _amountReceived - _reservedToken,
            _memo: "",
            _preferClaimedTokens: false
        });
    }
```

[JBXBuybackDelegate.sol#L334-L353](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L334-L353)

```diff
    function _mint(JBDidPayData calldata _data, uint256 _amount) internal {
-       IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

        // Mint to the beneficiary with the fc reserve rate
-       controller.mintTokensOf({
+       IJBController(jbxTerminal.directory().controllerOf(_data.projectId)).mintTokensOf({    
            _projectId: _data.projectId,
            _tokenCount: _amount,
            _beneficiary: _data.beneficiary,
            _memo: _data.memo,
            _preferClaimedTokens: _data.preferClaimedTokens,
            _useReservedRate: true
        });

        // Send the eth back to the terminal balance
        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );

        emit JBXBuybackDelegate_Mint(_data.projectId);
    }
```

### Test Gas Profile

#### snapshot
```
forge snapshot --diff
```

```diff
|   Test Name                        | Gas    |
|------------------------------------|--------|
-| test_mintIfPreferClaimedIsFalse() | 995415 |
+| test_mintIfPreferClaimedIsFalse() | 995402 |
|------------------------------------|--------|
|                                    |    -13 |

The saved gas is 13.
```
                

```diff
|   Test Name                   | Gas     |
|-------------------------------|---------|
-| test_mintIfSlippageTooHigh() | 1082925 |
+| test_mintIfSlippageTooHigh() | 1082888 |
|-------------------------------|---------|
|                               |     -37 |

The saved gas is 37.
```
                

```diff
|   Test Name                      | Gas     |
|----------------------------------|---------|
-| test_swapIfQuoteBetter(uint256) | 1114547 |
+| test_swapIfQuoteBetter(uint256) | 1114511 |
|----------------------------------|---------|
|                                  |     -36 |

The average saved gas is 36.
```
                

```diff
|   Test Name          | Gas     |
|----------------------|---------|
-| test_swapMultiple() | 1361806 |
+| test_swapMultiple() | 1361734 |
|----------------------|---------|
|                      |     -72 |

The saved gas is 72.
```
                

```diff
|   Test Name                       | Gas     |
|-----------------------------------|---------|
-| test_swapRandomAmountIn(uint256) | 1084099 |
+| test_swapRandomAmountIn(uint256) | 1084073 |
|-----------------------------------|---------|
|                                   |     -26 |

The average saved gas is 26.
```
                

```diff
|   Test Name                                              | Gas    |
|----------------------------------------------------------|--------|
-| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265670 |
+| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265647 |
|----------------------------------------------------------|--------|
|                                                          |    -23 |

The saved gas is 23.
```
                

```diff
|   Test Name                                  | Gas   |
|----------------------------------------------|-------|
-| testRevertIfSlippageIsTooMuchWhenSwapping() | 12291 |
+| testRevertIfSlippageIsTooMuchWhenSwapping() | 12280 |
|----------------------------------------------|-------|
|                                              |   -11 |

The saved gas is 11.
```
                

| Test                              | Name                                                    | Gas   |
|-----------------------------------|---------------------------------------------------------|-------|
| TestIntegrationJBXBuybackDelegate | test_mintIfPreferClaimedIsFalse()                       | -13   |
| TestIntegrationJBXBuybackDelegate | test_mintIfSlippageTooHigh()                            | -37   |
| TestIntegrationJBXBuybackDelegate | test_swapIfQuoteBetter(uint256)                         | -36   |
| TestIntegrationJBXBuybackDelegate | test_swapMultiple()                                     | -72   |
| TestIntegrationJBXBuybackDelegate | test_swapRandomAmountIn(uint256)                        | -26   |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | -23   |
| TestUnitJBXBuybackDelegate        | testRevertIfSlippageIsTooMuchWhenSwapping()             | -11   |
| --                                | --                                                      |       |
| Total                             |                                                         | -218  |

#### gas report
```
forge test --gas-report
```


|    | Contract             | Function name         | Gas avg   | Gas diff   |
|----|----------------------|-----------------------|-----------|------------|
| -  | JBXBuybackDelegate   | didPay                | 161926    |            |
| +  | JBXBuybackDelegate   | didPay                | 161101    | -825       |
| -  | JBXBuybackDelegate   | uniswapV3SwapCallback | 20125     |            |
| +  | JBXBuybackDelegate   | uniswapV3SwapCallback | 20114     | -11        |
| -  | JBETHPaymentTerminal | pay                   | 137702    |            |
| +  | JBETHPaymentTerminal | pay                   | 137695    | -7         |
| -- | --                   | --                    |           |            |
|    | Total                |                       |           | -843       |



## GAS-02 - Using alternative formulation in order to avoid to call variable twice

### Founded in [JBXBuybackDelegate.sol](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol)


[JBXBuybackDelegate.sol#L156](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156)

```
if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
```

[JBXBuybackDelegate.sol#L197](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L197)

```
uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
```


#### Description
Avoid to call _quote twice.

```
_quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)

is equivalent to

_quote * (1 - _slippage / SLIPPAGE_DENOMINATOR)
```

[JBXBuybackDelegate.sol#L156](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156)

```diff
-   if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
+   if (_tokenCount < _quote * (1 - _slippage / SLIPPAGE_DENOMINATOR)) {
```

[JBXBuybackDelegate.sol#L197](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L197)

```diff
-   uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
+   uint256 _minimumReceivedFromSwap = _quote * (1 - _slippage / SLIPPAGE_DENOMINATOR);
```

### Test Gas Profile

#### snapshot
```
forge snapshot --diff
```
```diff
|   Test Name                        | Gas    |
|------------------------------------|--------|
-| test_mintIfPreferClaimedIsFalse() | 995415 |
+| test_mintIfPreferClaimedIsFalse() | 995409 |
|------------------------------------|--------|
|                                    |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                   | Gas     |
|-------------------------------|---------|
-| test_mintIfSlippageTooHigh() | 1082925 |
+| test_mintIfSlippageTooHigh() | 1082919 |
|-------------------------------|---------|
|                               |      -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                  | Gas    |
|----------------------------------------------|--------|
-| test_mintIfWeightGreatherThanPrice(uint256) | 958102 |
+| test_mintIfWeightGreatherThanPrice(uint256) | 958099 |
|----------------------------------------------|--------|
|                                              |     -3 |

The average saved gas is 3.
```
                

```diff
|   Test Name                      | Gas     |
|----------------------------------|---------|
-| test_swapIfQuoteBetter(uint256) | 1114547 |
+| test_swapIfQuoteBetter(uint256) | 1114541 |
|----------------------------------|---------|
|                                  |      -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name          | Gas     |
|----------------------|---------|
-| test_swapMultiple() | 1361806 |
+| test_swapMultiple() | 1361794 |
|----------------------|---------|
|                      |     -12 |

The saved gas is 12.
```
                

```diff
|   Test Name                       | Gas     |
|-----------------------------------|---------|
-| test_swapRandomAmountIn(uint256) | 1084099 |
+| test_swapRandomAmountIn(uint256) | 1084093 |
|-----------------------------------|---------|
|                                   |      -6 |

The average saved gas is 6.
```
                

```diff
|   Test Name                                                 | Gas    |
|-------------------------------------------------------------|--------|
-| testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() | 189895 |
+| testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens() | 189892 |
|-------------------------------------------------------------|--------|
|                                                             |     -3 |

The saved gas is 3.
```
                

```diff
|   Test Name                                              | Gas    |
|----------------------------------------------------------|--------|
-| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265670 |
+| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265664 |
|----------------------------------------------------------|--------|
|                                                          |     -6 |

The saved gas is 6.
```
                

```diff
|   Test Name                                                    | Gas    |
|----------------------------------------------------------------|--------|
-| testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) | 190552 |
+| testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) | 190549 |
|----------------------------------------------------------------|--------|
|                                                                |     -3 |

The average saved gas is 3.
```
                

| Test                              | Name                                                          | Gas   |
|-----------------------------------|---------------------------------------------------------------|-------|
| TestIntegrationJBXBuybackDelegate | test_mintIfPreferClaimedIsFalse()                             | -6    |
| TestIntegrationJBXBuybackDelegate | test_mintIfSlippageTooHigh()                                  | -6    |
| TestIntegrationJBXBuybackDelegate | test_mintIfWeightGreatherThanPrice(uint256)                   | -3    |
| TestIntegrationJBXBuybackDelegate | test_swapIfQuoteBetter(uint256)                               | -6    |
| TestIntegrationJBXBuybackDelegate | test_swapMultiple()                                           | -12   |
| TestIntegrationJBXBuybackDelegate | test_swapRandomAmountIn(uint256)                              | -6    |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateMintIfPreferenceIsNotToClaimTokens()    | -3    |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateSwapIfPreferenceIsToClaimTokens()       | -6    |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateWhenQuoteIsLowerThanTokenCount(uint256) | -3    |
| --                                | --                                                            |       |
| Total                             |                                                               | -51   |

#### gas report
```
forge test --gas-report
```

|    | Contract                          | Function name     | Gas avg   | Gas diff   |
|----|-----------------------------------|-------------------|-----------|------------|
| -  | JBXBuybackDelegate                | didPay            | 161926    |            |
| +  | JBXBuybackDelegate                | didPay            | 161124    | -802       |
| -  | JBXBuybackDelegate                | payParams         | 8271      |            |
| +  | JBXBuybackDelegate                | payParams         | 8268      | -3         |
| -  | JBETHPaymentTerminal              | pay               | 137702    |            |
| +  | JBETHPaymentTerminal              | pay               | 137698    | -4         |
| -  | JBSingleTokenPaymentTerminalStore | recordPaymentFrom | 44640     |            |
| +  | JBSingleTokenPaymentTerminalStore | recordPaymentFrom | 44637     | -3         |
| -- | --                                | --                |           |            |
|    | Total                             |                   |           | -812       |




## GAS-03 - Avoid to define useless variable

### Founded in [JBXBuybackDelegate.sol](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol)


[JBXBuybackDelegate.sol#L258-L326](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326)

```
function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
    internal
    returns (uint256 _amountReceived)
{
    // Pass the token and min amount to receive as extra data
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

    // The amount to send to the beneficiary
    uint256 _nonReservedToken = PRBMath.mulDiv(
        _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
    );

    // The amount to add to the reserved token
    uint256 _reservedToken = _amountReceived - _nonReservedToken;

    // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
    if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

    // If there are reserved token, add them to the reserve
    if (_reservedToken != 0) {
        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

        // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
        controller.burnTokensOf({
            _holder: address(this),
            _projectId: _data.projectId,
            _tokenCount: _reservedToken,
            _memo: "",
            _preferClaimedTokens: true
        });

        // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
        controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amountReceived,
            _beneficiary: address(this),
            _memo: _data.memo,
            _preferClaimedTokens: false,
            _useReservedRate: true
        });

        // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
        uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

        if (_nonReservedTokenInContract != 0) {
            controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
                _tokenCount: _nonReservedTokenInContract,
                _memo: "",
                _preferClaimedTokens: false
            });
        }
    }

    emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
}
```


#### Description
You could avoid to define _nonReservedTokenInContract.

```
uint256 _reservedToken = _amountReceived - _nonReservedToken;
```
and

```
uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
```

If we ```replace _reservedToken``` in ```_nonReservedTokenInContract```, we obtain:

```
uint256 _nonReservedTokenInContract = _amountReceived - (_amountReceived - _nonReservedToken);
```

which is equivalent to 

```
uint256 _nonReservedTokenInContract = _nonReservedToken
```

So we could avoid to use ```_nonReservedTokenInContract```, and use ```_nonReservedToken``` instead 

[JBXBuybackDelegate.sol#L258-L326](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326)

```diff
function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
    internal
    returns (uint256 _amountReceived)
{
    // Pass the token and min amount to receive as extra data
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

    // The amount to send to the beneficiary
    uint256 _nonReservedToken = PRBMath.mulDiv(
        _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
    );

    // The amount to add to the reserved token
    uint256 _reservedToken = _amountReceived - _nonReservedToken;

    // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
    if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

    // If there are reserved token, add them to the reserve
    if (_reservedToken != 0) {
        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

        // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
        controller.burnTokensOf({
            _holder: address(this),
            _projectId: _data.projectId,
            _tokenCount: _reservedToken,
            _memo: "",
            _preferClaimedTokens: true
        });

        // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
        controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amountReceived,
            _beneficiary: address(this),
            _memo: _data.memo,
            _preferClaimedTokens: false,
            _useReservedRate: true
        });

        // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
-       uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

-       if (_nonReservedTokenInContract != 0) {
+       if (_nonReservedToken != 0) {
            controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
-               _tokenCount: _nonReservedTokenInContract,
+               _tokenCount: _nonReservedToken,
                _memo: "",
                _preferClaimedTokens: false
            });
        }
    }

    emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
}
```

### Test Gas Profile

#### snapshot
```
forge snapshot --diff
```
```diff
|   Test Name                      | Gas     |
|----------------------------------|---------|
-| test_swapIfQuoteBetter(uint256) | 1114547 |
+| test_swapIfQuoteBetter(uint256) | 1114462 |
|----------------------------------|---------|
|                                  |     -85 |

The average saved gas is 85.
```
                

```diff
|   Test Name          | Gas     |
|----------------------|---------|
-| test_swapMultiple() | 1361806 |
+| test_swapMultiple() | 1361636 |
|----------------------|---------|
|                      |    -170 |

The saved gas is 170.
```
                

```diff
|   Test Name                                              | Gas    |
|----------------------------------------------------------|--------|
-| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265670 |
+| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265585 |
|----------------------------------------------------------|--------|
|                                                          |    -85 |

The saved gas is 85.
```
                

| Test                              | Name                                                    | Gas   |
|-----------------------------------|---------------------------------------------------------|-------|
| TestIntegrationJBXBuybackDelegate | test_swapIfQuoteBetter(uint256)                         | -85   |
| TestIntegrationJBXBuybackDelegate | test_swapMultiple()                                     | -170  |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | -85   |
| --                                | --                                                      |       |
| Total                             |                                                         | -340  |

#### gas report
```
forge test --gas-report
```

|    | Contract             | Function name   | Gas avg   | Gas diff   |
|----|----------------------|-----------------|-----------|------------|
| -  | JBXBuybackDelegate   | didPay          | 161926    |            |
| +  | JBXBuybackDelegate   | didPay          | 161088    | -838       |
| -  | JBETHPaymentTerminal | pay             | 137702    |            |
| +  | JBETHPaymentTerminal | pay             | 137679    | -23        |
| -- | --                   | --              |           |            |
|    | Total                |                 |           | -861       |





## GAS-04 - Declaring, using and modifying variable only where it needs

### Founded in [JBXBuybackDelegate.sol](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol)


[JBXBuybackDelegate.sol#L195-L208](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L195-L208)

```
// The minimum amount of token received if swapping
(,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

// Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
if (_data.preferClaimedTokens) {
    // Try swapping
    uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

    // If swap failed, mint instead, with the original weight + add to balance the token in
    if (_amountReceived == 0) _mint(_data, _tokenCount);
} else {
    _mint(_data, _tokenCount);
}
```


#### Description
You should manipulate variable only when it needs.
For example, in if/else, if you use variable only in one branch, you could declare and manipulate variable only in that branch.
Of course, you can do it only when the variable's scope is the function and the variable is not used by other elements outside branch.

[JBXBuybackDelegate.sol#L195-L208](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L195-L208)

```DIFF
-   // The minimum amount of token received if swapping
-   (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
-   uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

    // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
    if (_data.preferClaimedTokens) {

+       // The minimum amount of token received if swapping
+       (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
+       uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

        // Try swapping
        uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

        // If swap failed, mint instead, with the original weight + add to balance the token in
        if (_amountReceived == 0) _mint(_data, _tokenCount);
    } else {
        _mint(_data, _tokenCount);
    }
```

### Test Gas Profile

#### snapshot
```
forge snapshot --diff
```
```diff
|   Test Name                        | Gas    |
|------------------------------------|--------|
-| test_mintIfPreferClaimedIsFalse() | 995415 |
+| test_mintIfPreferClaimedIsFalse() | 994813 |
|------------------------------------|--------|
|                                    |   -602 |

The saved gas is 602.
```
                

```diff
|   Test Name                   | Gas     |
|-------------------------------|---------|
-| test_mintIfSlippageTooHigh() | 1082925 |
+| test_mintIfSlippageTooHigh() | 1082913 |
|-------------------------------|---------|
|                               |     -12 |

The saved gas is 12.
```
                

```diff
|   Test Name                      | Gas     |
|----------------------------------|---------|
-| test_swapIfQuoteBetter(uint256) | 1114547 |
+| test_swapIfQuoteBetter(uint256) | 1114535 |
|----------------------------------|---------|
|                                  |     -12 |

The average saved gas is 12.
```
                

```diff
|   Test Name          | Gas     |
|----------------------|---------|
-| test_swapMultiple() | 1361806 |
+| test_swapMultiple() | 1361782 |
|----------------------|---------|
|                      |     -24 |

The saved gas is 24.
```
                

```diff
|   Test Name                       | Gas     |
|-----------------------------------|---------|
-| test_swapRandomAmountIn(uint256) | 1084099 |
+| test_swapRandomAmountIn(uint256) | 1084087 |
|-----------------------------------|---------|
|                                   |     -12 |

The average saved gas is 12.
```
                

```diff
|   Test Name                                              | Gas    |
|----------------------------------------------------------|--------|
-| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265670 |
+| testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | 265658 |
|----------------------------------------------------------|--------|
|                                                          |    -12 |

The saved gas is 12.
```
                

| Test                              | Name                                                    | Gas   |
|-----------------------------------|---------------------------------------------------------|-------|
| TestIntegrationJBXBuybackDelegate | test_mintIfPreferClaimedIsFalse()                       | -602  |
| TestIntegrationJBXBuybackDelegate | test_mintIfSlippageTooHigh()                            | -12   |
| TestIntegrationJBXBuybackDelegate | test_swapIfQuoteBetter(uint256)                         | -12   |
| TestIntegrationJBXBuybackDelegate | test_swapMultiple()                                     | -24   |
| TestIntegrationJBXBuybackDelegate | test_swapRandomAmountIn(uint256)                        | -12   |
| TestUnitJBXBuybackDelegate        | testDatasourceDelegateSwapIfPreferenceIsToClaimTokens() | -12   |
| --                                | --                                                      |       |
| Total                             |                                                         | -674  |

#### gas report
```
forge test --gas-report
```

|    | Contract             | Function name   | Gas avg   | Gas diff   |
|----|----------------------|-----------------|-----------|------------|
| -  | JBXBuybackDelegate   | didPay          | 161926    |            |
| +  | JBXBuybackDelegate   | didPay          | 161832    | -94        |
| -  | JBETHPaymentTerminal | pay             | 137702    |            |
| +  | JBETHPaymentTerminal | pay             | 137698    | -4         |
| -- | --                   | --              |           |            |
|    | Total                |                 |           | -98        |