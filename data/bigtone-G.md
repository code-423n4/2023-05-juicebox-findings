# Report

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | No need to calculate the sqrtPriceLimitX96 | 1 |
| [GAS-2](#GAS-2) | Importing the TickMath library is unnecessary. | 1 |
| | 

### <a name="GAS-1"></a>[GAS-1] No need to calculate the sqrtPriceLimitX96
Instead, you can consider sqrtPriceLimitX96 as 0 without any restriction on the rate. See uniswap v3 pool [source](https://github.com/Uniswap/v3-sdk/blob/08a7c05/src/entities/pool.ts#L220).

*Instances (1)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

267:     sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
=>
267:     sqrtPriceLimitX96: 0,
```


### <a name="GAS-2"></a>[GAS-2] Importing the TickMath library is unnecessary.
The import of the TickMath library is not required from [GAS-1](#GAS-1).

*Instances (1)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

22:     import "@uniswap/v3-core/contracts/libraries/TickMath.sol";

```

