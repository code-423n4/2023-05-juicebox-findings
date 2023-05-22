### Gas Optimizations

| Number | Issue | Instances | Min Gas Savings (Est.) |
| :-: | :-- | :-: | :-: |
| [G-01] | State variables should be cached | 3 | 300 |
| [G-02] | Using bools for storage incurs overhead | 1 | 100 |

#### [G-01] State variables should be cached

The instances below point to the second plus access of a state variable within a function. Caching of a state variable replaces each `Gwarmaccess` (100 gas) with a much cheaper stack read.

Saves 100 gas per instance.

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

// _projectTokenIsZero in uniswapV3SwapCallback()
224:    uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
225:    uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

// _projectTokenIsZero in _swap()
265:    zeroForOne: !_projectTokenIsZero,
267:    sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
271:    _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
```

#### [G-02] Using bools for storage incurs overhead

Use `uint256(1)` and `uint256(2)` for `true` and `false` to avoid a `Gwarmaccess` (100 gas), and to avoid `Gsset` (20,000 gas) when changing from `false` to `true`, after having been `true` in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

```solidity
File: JBXBuybackDelegate.sol

63:    bool private immutable _projectTokenIsZero;
```
