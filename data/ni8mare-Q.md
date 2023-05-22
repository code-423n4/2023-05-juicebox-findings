# Non-Critical Issues

## [NC-1] Spelling error in JBXBuybackDelegate
`control` is spelt as `controle` in line [214](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L214) in the comment for the function `uniswapV3SwapCallback`.
`Slippage controle is achieved here` should be changed to `Slippage control is achieved here`.

## [NC-2] There is no need to use Ownable
`Ownable` contract by Openzeppelin is being inherited by `JBXBuybackDelegate` contract, but the functions defined in the contract do not make use of the `onlyOwner()` modifier. Hence, the Ownable contract can be removed from JBXBuybackDelegate.