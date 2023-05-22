### [LOW] Should check parameters are not zero in the constructor

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L123

We should check projectToken, pool, and jbxTerminal are not zeros for sanity.

Even further, we can call relative read functions like `totalSupply()>0` for `_weth` and `supportsInterface` for `_jbxTerminal`.

### [non-critical] Ownable Module is not used

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L39

JBXBuybackDelegate inherits openzepplin's Ownable module, but there are no onlyOwner modifiers in the code. We should remove this module.