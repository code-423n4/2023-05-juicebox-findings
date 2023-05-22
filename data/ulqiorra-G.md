Private variables `mintedAmount` and `reservedRate` are set to 1 in the constructor and after each didPay() call:
```
uint256 private mintedAmount = 1;
...
uint256 private reservedRate = 1;
...
uint256 _tokenCount = mintedAmount;
mintedAmount = 1;
...
uint256 _reservedRate = reservedRate;
reservedRate = 1;
```
* https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L106
* https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L113
* https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L189
* https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L193

These private variables are expected to be set in `payParams()` only and to be used in `didPay()` only after the `payParams()` is called. Thus, it is safe to remove all lines which reset there variables to `1`, since those values are expected to be rewritten by `payParams()` anyway.