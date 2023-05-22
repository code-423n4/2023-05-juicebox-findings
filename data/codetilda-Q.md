# [L‑01] Return value of weth.transfer() not checked

weth.transfer() will return true if transfer done without revert, but in the code we don't check it.

[JBXBuybackDelegate.sol#LL232C3-L232C3](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL232C3-L232C3)

        // Wrap and transfer the weth to the pool
        weth.deposit{value: _amountToSend}();
        weth.transfer(address(pool), _amountToSend); // @audit not checked return value
    }
