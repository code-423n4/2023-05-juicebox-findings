- redundant variable `_nonReservedTokenInContract`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

Variable `_nonReservedTokenInContract` above is redundant, can use `_nonReservedToken` declared on https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L278

- Unused revert codes

In `uniswapV3SwapCallback` function, various revert codes are used.
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L218
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L228

However, since the swap is called inside try/catch, those won't be used.
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L272 

Can just revert instead of with those revert codes if there's no intention to retrieve/process revert codes.
