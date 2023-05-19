# Findings Summary

| ID     | Title                                    | Severity     |
| ------ | ---------------------------------------- | ------------ |
| [N-01] | _nonReservedToken duplicate calculation  | Non-critical |
| [N-02] | Ownable has not been used                | Non-critical |


# Detailed Findings

# [N-01] _nonReservedToken duplicate calculation

## Link

[_nonReservedTokenInContract](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#LL312C23-L312C23)

## Description

`_nonReservedTokenInContract` equals `_nonReservedToken`, do not need to duplicate calculate.   

## Recommendations

Use `_nonReservedToken`

# [N-02] Ownable has not been used

## Link

[Ownable](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L39)

## Description

`Ownable` has not been used, can remove.    

## Recommendations

Remove `Ownable`

