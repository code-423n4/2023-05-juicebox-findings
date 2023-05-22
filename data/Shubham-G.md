| |Issue|Instances|
|-|:-|:-:|
| [GAS-01](#GAS-01) | Inverting the condition of an if-else-statement saves gas | 1 |
| [GAS-02](#GAS-02) | bytes constants are more efficient than string constants | 2 |


## [GAS-01] Inverting the condition of an if-else-statement saves gas

Flipping the true and false blocks instead saves gas. Average gas is reduced which is evident in the gas report. 

```solidity

L:267      sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
```
Recommended

```solidity
L:267       sqrtPriceLimitX96: !_projectTokenIsZero ? TickMath.MIN_SQRT_RATIO + 1 : TickMath.MAX_SQRT_RATIO - 1,
```
| |Deployment Cost |Deployment Size| Avg |
|-|:-|:-:|:-:|
|Before| 1297315 | 6757 | 160925 |
|After| 1290293 | 6715 | 160130 |

## [GAS-02] bytes constants are more efficient than string constants

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
L:147        string memory memo

L:238        string memory memo
```

     