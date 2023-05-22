# NC-01 Redundant variable returned in `_swap()`
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L274

The value returned must be zero, this can be made clearer with `return 0`

# NC-02 Redundant calculation
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

We can remove this line of code and use `_nonReservedToken` instead of `_nonReservedTokenInContract`
