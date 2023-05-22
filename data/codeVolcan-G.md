## IERC20 interface imported multiple times.

### Proof of Concept:

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L16

**IWETH9.sol is importing the IERC20 interface so there's no need to import it in main contract again.**

## ............................................................

## Unused function. function redeemParams() is not used anywhere.

### Proof Of Concept: 

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

**Consider using or remove it.**

## ............................................................

## unused imported contract.Ownable contract imported from oprnzeppelin is not used anywhere.

### Proof Of Concept: 

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L15

**Consider using or remove it.**