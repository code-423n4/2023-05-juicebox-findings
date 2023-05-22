### [Low-01] Multiple calls to `payParams()` will cause `override` to state variables `mintedAmount` and `reservedRate`
*Instances(1)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144
```

### [Low-02] Instead of `<` it should `<=` in `if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR))` condition check
```solidity
    function payParams(JBPayParamsData calldata _data) // @audit-issue multi time call to this function will cuse state variable to oveeride
        external
        override
        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
    {
        // Find the total number of tokens to mint, as a fixed point number with 18 decimals
        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18); 

        // Unpack the quote from the pool, given by the frontend
        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

        // If the amount swapped is bigger than the lowest received when minting, use the swap pathway 

        // @audit-info _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR) == _minimumReceivedFromSwap
        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) { // @audit should be <=
            // Pass the quote and reserve rate via a mutex
            mintedAmount = _tokenCount; 
            reservedRate = _data.reservedRate;

            // Return this delegate as the one to use, and do not mint from the terminal
            delegateAllocations = new JBPayDelegateAllocation[](1);
            delegateAllocations[0] =
                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value}); // @audit gas optimized

            return (0, _data.memo, delegateAllocations); // @audit-info 0 => for swapping, or the one corresponding to the reserved token to mint if minting
        }
```
*Instances(1)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156
```

### [Low-03] Some Functions should be marked `payable` as they handling `ETH`
```solidity
function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override { // @audit payable
        // Check if this is really a callback
        if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

        // Unpack the data
        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

        // Assign 0 and 1 accordingly
        uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
        uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

        // Revert if slippage is too high
        if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

        // Wrap and transfer the weth to the pool
        weth.deposit{value: _amountToSend}(); 
        weth.transfer(address(pool), _amountToSend);
    }

```
And
```solidity
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
        jbxTerminal.addToBalanceOf{value: _data.amount.value}( // @audit-issue is payable required, is transaction fail revert
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );

        emit JBXBuybackDelegate_Mint(_data.projectId);
    }
```
*Instances(2)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L216
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L334
```

### [NC-01] Empty code block should do something, Atleast emit something when ever called.
*Instances(1)*
```solidity
file:: juice-buyback/contracts/JBXBuybackDelegate.sol
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L237
```