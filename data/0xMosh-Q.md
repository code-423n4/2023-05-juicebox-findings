## [N-01]Typos 
Typos .
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L214
```
 * @dev    Slippage controle is achieved here //@audit typos- "controle" -> control
```
## [N-02] `_nonReservedToken` is calculated once . No need to calculate again .
`_nonReservedToken` is calculated in line [278](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L278)
```
 uint256 _nonReservedToken = PRBMath.mulDiv(
            _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
        );
```
no need to calculate again in line[312](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312) . Both of them refers to the same value .
```
 uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
```


