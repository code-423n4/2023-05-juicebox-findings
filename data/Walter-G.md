yul language (inline assembly) will save gas in different situations:
first,the calculation to get the _amountReceived and the _amountToSend variables at line 224,225
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L225
this could be re-write as:
```
 uint256 _amountReceived=0;
 uint256 _amountToSend=0;
    bool i = _projectTokenIsZero;
    assembly {
      switch i
      case true {
        _amountReceived := sub(0, amount0Delta)
        _amountToSend := amount1Delta
      }
      case false {
        _amountReceived := sub(0, amount1Delta)
        _amountToSend := amount0Delta
      }
    }
```
this will save 44gas

another situation where we could use the yul language is in line 197,and here we'll save 172gas,here's how:
```
        uint256 _minimumReceivedFromSwap;
        assembly{
            _minimumReceivedFromSwap := sub(_quote,div(mul(_quote,_slippage),SLIPPAGE_DENOMINATOR))
        }
```
