# Summary

| Id | Title |
| -- | ----- |
| 1 | Uniswap V3 pool is not validated |
| 2 | Unnecessary Ownable import |
| 3 | Typo in comments |

# 1. Uniswap V3 pool is not validated

https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L125

## Detail
Uniswap V3 pool is set in the constructor without any validation. If its address is set to wrong or malicious pool with wrong pair of tokens, it could result in wrong token out or wrong amout out.

## Recommendation
Consider using Uniswap V3 Factory to get the address of the pool of WETH and project token.

# 2. Unnecessary Ownable import

https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L39

## Detail
Contract imported Ownable but did not use anywhere in the codebase.

## Recommendation
Remove unnecessary Ownable import.

# 3. Typo in comments
```diff
- pay beneficiary
+ payees

- the project weigh
+ the project weight
```

