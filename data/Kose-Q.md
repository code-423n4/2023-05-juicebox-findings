# QA Report
## Issue List
| Number |Issues Details|Instances|
|:--:|:-------|:--:|
|[N-01]|WETH address definition can be use directly| 2 |
|[N-02]| Immutables should follow the naming convention ```UPPERCASE_WITH_UNDERSCORES```| 5 |
|[N-03]|Important event is missing parameters | 1 |

Total 8 instance in 3 issues

# Findings

## [N-01] WETH address definition can be use directly

WETH is a wrap Ether contract with a specific address in the Ethereum network, giving the option to define it may cause false recognition, it is healthier to define it directly.

[JBXBuybackDelegate.sol#L95](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L95)
```solidity
IWETH9 public immutable weth;
```

Advantages of defining a specific contract directly:

It reduces gas, prevents incorrect argument definitions, prevents execution on a different blockchain and mitigates re-signature issues.

### Recommendation
```solidity
address private constant WETH = 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2;
```

1 file 1 instance


## [L-02]  Immutables should follow the naming convention ```UPPERCASE_WITH_UNDERSCORES```

```solidity
63:   bool private immutable _projectTokenIsZero;
...
79:   IERC20 public immutable projectToken;
...
85:   IUniswapV3Pool public immutable pool;
...
90:   IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;
...
95:   IWETH9 public immutable weth;
```
1 file 5 instances

## [NC-03] Important event is missing parameters

In the ```JBXBuybackDelegate_Mint``` event, there is only one parameter which is ```projectId```. 
```solidity
event JBXBuybackDelegate_Mint(uint256 projectId);
```
But it is crucial for listeners to reach information about amount that is minted, without this parameter, emit does not serve the purpose.

### Recommendation

```solidity
event JBXBuybackDelegate_Mint(uint256 projectId, uint256 amountMinted);
```

1 file 1 instance
