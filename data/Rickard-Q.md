# [L-01] Use the safe variant and `ERC721.mint`
## Lines of code
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L205](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L205)       
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L207](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L207)
## Vulnerability details
### Impact
`.mint` won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset.     
      
OpenZeppelin [recommendation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L277) is to use the safe variant of `_mint`.     
## Tools Used
Manual review
## Recommended mitigation steps
Replace `_mint()` with `_safeMint()`.
# [N-01] Use underscores for number literals
## Lines of code
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68)
## Vulnerability details
### Impact
````solidity
juice-buyback/contracts/JBXBuybackDelegate.sol

68:   uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
````
## Tools Used
Manual review
## Recommended mitigation steps
````diff
- 68:   uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
+ 68:   uint256 private constant SLIPPAGE_DENOMINATOR = 10_000;
````