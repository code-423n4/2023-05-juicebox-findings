## PRBMath is not necessary
These calculations would not realistically overflow and so may equivalently be calculated as
```diff
- uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
+ uint256 _tokenCount = _data.amount.value * _data.weight / 10 ** 18;
```
(https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L150)
```diff
- uint256 _nonReservedToken = PRBMath.mulDiv(
-     _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
- );
+ uint256 _nonReservedToken = _amountReceived * (JBConstants.MAX_RESERVED_RATE - _reservedRate) / JBConstants.MAX_RESERVED_RATE;
```
(https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L278-L280)
(Regarding the latter also see below.)


## Calculation of already calculated value
[`uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;`](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L312)
is already calculated at [L278-L280](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L278-L280), since `_amountReceived - _reservedToken == _nonReservedToken`. Simply reuse `_nonReservedToken`;

## Reverse calculation of `_nonReservedToken` and `_reservedToken`
The order in which [`_nonReservedToken` and `_reservedToken` are calculated](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L278-L283) can be changed from
```js
// The amount to send to the beneficiary
uint256 _nonReservedToken = PRBMath.mulDiv(
    _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
);

// The amount to add to the reserved token
uint256 _reservedToken = _amountReceived - _nonReservedToken;
```
to
```js
// The amount to add to the reserved token
uint256 _reservedToken = PRBMath.mulDiv(
    _amountReceived, _reservedRate, JBConstants.MAX_RESERVED_RATE
);

// The amount to send to the beneficiary
uint256 _nonReservedToken = _amountReceived - _reservedToken;
```
saving a subtraction. This means that the rounding is also reversed, but there seems to be no preference for either way.