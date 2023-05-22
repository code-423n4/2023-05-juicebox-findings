
# Summary

## Gas Optimization

| No     | Issue  | Instances |
| ------ | --------- | --- |
| [G-01] |  Avoid using try/catch blocks:| 2  | - |
| [G-02] |  Make 3 event parameters indexed when possible.  | 2  | - |
| [G-03] |  Use hardcode address instead address(this).| 4  | - |
| [G-04] |  Remove unnecessary code to save gas. .| 1  | - |


### [G-01] Avoid using try/catch blocks:

 try/catch blocks can be expensive in terms of gas cost because they require additional storage and computations. If possible, use conditional statements to handle errors instead of try/catch blocks.

```solidity
file:       JBXBuybackDelegate.sol
263         try pool.swap({
            recipient: address(this),
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(_data.amount.value),
            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
            data: abi.encode(_minimumReceivedFromSwap)
272        catch {
            // implies _amountReceived = 0 -> will later mint when back in didPay
            return _amountReceived;
        }
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL263C1-L268C55
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL272C11-L275C10

## Recommendation Code instead of try

```solidity

function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
    internal
    returns (uint256 _amountReceived)
{
    // Get the input token and output token addresses for the Uniswap pool
    address inputToken = _projectTokenIsZero ? pool.token1() : pool.token0();
    address outputToken = _projectTokenIsZero ? pool.token0() : pool.token1();
    
    // Get the input and output token amounts for the swap
    uint256 inputAmount = _data.amount.value;
    uint256 outputAmount = _minimumReceivedFromSwap;
    
    // Perform the swap
    (bool success, ) = address(pool).call(
        abi.encodeWithSelector(
            pool.swap.selector,
            address(this),
            !_projectTokenIsZero,
            int256(inputAmount),
            int256(0),
            _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO +1,
            abi.encode(outputAmount)
        )
    );
    
    // Check if the swap was successful
    if (!success) {
        revert JuicBuyback_Unauthorized();
    }
    
    // Get the output token amount received from the swap
    uint256 outputAmountReceived = _projectTokenIsZero ? IERC20(outputToken).balanceOf(address(this)) : IERC20(outputToken).balanceOf(address(this));
    
    // Check if the output token amount received is less than the minimum amount required
    if (outputAmountReceived < outputAmount) {
        revert JuiceBuyback_MaximumSlippage();
    }
    
    // Return the output token amount received
  
```
## Recommendation Code instead of catch


```solidity

function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
    internal
    returns (uint256 _amountReceived)
{
    // Get the input token and output token addresses for the Uniswap pool
    address inputToken = _projectTokenIsZero ? pool.token1() : pool.token0();
    address outputToken = _projectTokenIsZero ? pool.token0() : pool.token1();
    
    // Get the input and output token amounts for the swap
    uint256 inputAmount = _data.amount.value;
    uint256 outputAmount = _minimumReceivedFromSwap;
    
    // Perform the swap
    (bool success, ) = address(pool).call(
        abi.encodeWithSelector(
            pool.swap.selector,
            address(this),
            !_projectTokenIsZero,
            int256(inputAmount),
            int256(0),
            _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO +1,
            abi.encode(outputAmount)
        )
    );
    
    // Check if the swap was successful
    if (!success) {
        // implies _amountReceived = 0 -> will later mint when back in didPay
        return 0;
    }
    
    // Get the output token amount received from the swap
    uint256 outputAmountReceived = _projectTokenIsZero ? IERC20(outputToken).balanceOf(address(this)) : IERC20(outputToken).balanceOf(address(this));
    
    // Check if the output token amount received is less than the minimum amount required
    if (outputAmountReceived < outputAmount) {
        revert JuiceBuyback_MaximumSlippage();
    }
    
    // Return the output token amount received
    return outputAmountReceived;
}

```


### [G-02]  Make 3 event parameters indexed when possible. 

```solidity
file:     JBXBuybackDelegate.sol
53        event JBXBuybackDelegate_Swap(uint256 projectId, uint256 amountEth, uint256 amountOut);
54        event JBXBuybackDelegate_Mint(uint256 projectId);
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L53
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL54C5-L54C54
##   recomanded code is 

```solidity
 event JBXBuybackDelegate_Swap(uint256 index projectId, uint256 index amountEth, uint256 index amountOut);
 event JBXBuybackDelegate_Mint(uint256 index projectId);
```



### [G-03] Use hardcode address instead address(this).

```solidity
file:    JBXBuybackDelegate.sol
264      recipient: address(this),
294      _holder: address(this),
305      _beneficiary: address(this),
316      _holder: address(this),
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL264C13-L264C38
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL294C17-L294C40
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL305C17-L305C45
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL316C21-L316C44



### [G-04]   Remove unnecessary code to save gas.

```solidity
file:   JBXBuybackDelegate.sol
235     function redeemParams(JBRedeemParamsData calldata _data)
        external
        override
        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
    {}

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL235C1-L239C7

