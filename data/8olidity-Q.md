## [LOW-1]Weth's transfer() return value is not checked


`uniswapV3SwapCallback` will call the transfer function of weth

```solidity
weth.transfer(address(pool), _amountToSend);
```

But according to the code of weth, the transfer function has a return value. But the contract doesn't check

```solidity
function transfer(address dst, uint wad) public returns (bool) {
        return transferFrom(msg.sender, dst, wad);
    }
```