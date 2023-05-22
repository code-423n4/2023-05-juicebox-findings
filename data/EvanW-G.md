[G-01] use `struct` for `mintedAmount` and `reservedRate`

Obviously the `mintedAmount` and `reservedRate` will never hit the max. of uint256 or even uint128. And these 2 variables are always updated at the same time.
Thus, we can restructure it as 2 uint128 variable and store in `struct` for saving gas.


Suggestion as an example:

```solidity
struct PrivateProperties {
        uint128 mintedAmount;
        uint128 reservedRate;
    }

PrivateProperties private _privateProperties = PrivateProperties({
      mintedAmount: 1,
      reservedRate: 1
  });
```