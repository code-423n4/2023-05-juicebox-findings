### _nonReservedTokenInContract is duplicated with _nonReservedToken

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

`_nonReservedTokenInContract` is calculated as:

```solidity
            // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
```

Which is exactly `_nonReservedToken`:

```solidity
        // The amount to add to the reserved token
        uint256 _reservedToken = _amountReceived - _nonReservedToken;

        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
```

Use _nonReservedToken can save like ~100 gas: `didpay`

Before:

```
| contracts/JBXBuybackDelegate.sol:JBXBuybackDelegate contract |                 |        |        |        |         |
|--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                              | Deployment Size |        |        |        |         |
| 1310928                                                      | 6825            |        |        |        |         |
| Function Name                                                | min             | avg    | median | max    | # calls |
| didPay                                                       | 54376           | 163964 | 148968 | 244808 | 7       |
```

Afer:
```
| contracts/JBXBuybackDelegate.sol:JBXBuybackDelegate contract |                 |        |        |        |         |
|--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                              | Deployment Size |        |        |        |         |
| 1307928                                                      | 6810            |        |        |        |         |
| Function Name                                                | min             | avg    | median | max    | # calls |
| didPay                                                       | 54308           | 163925 | 148900 | 244740 | 7       |
```
