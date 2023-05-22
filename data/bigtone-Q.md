# Report

## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Ownable is not used | 1 |
| [L-2](#L-2) | Importing the Ownable is unnecessary. | 1 |
| | 

### <a name="L-1"></a>[L-1] Ownable is not used
Since the "onlyOwner" modifier is not utilized in any part of the code, it is unnecessary to include the "Ownable" contract.

*Instances (1)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

39:     contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {
=>
39:     contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback {
```


### <a name="L-2"></a>[L-2] Importing the TickMath library is unnecessary.
The import of the Ownable is not required from [L-1](#L-1).

*Instances (1)*:
```solidity
File: contracts/JBXBuybackDelegate.sol

15:     import "@openzeppelin/contracts/access/Ownable.sol";
```

