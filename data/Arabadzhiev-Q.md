# [L-01] Unnecessary payable function

The `didPay` function is currently declared as payable. However, it doesn't handle any ether that may be sent to it. This can result in users mistakenly sending ether to it and losing it forever. Consider making the function non-payable.

## Lines of code 
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L183

# [L-02] The contract inherits Ownable but doesn't use any of its functionality

The contract currently inherits the Ownable contract from OpenZeppelin. However, as of now, it does not make use of any of its functionality. Consider removing this inheritance relationship, or make use of it.


## Lines of code
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L39

# [N-01] Function with empty implementation

The body of the `redeemParams` function is currently empty. Consider segregating the function into a separate interface, so that it doesn't have to implemented where it's not needed.

## Lines of code

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L235

# [N-02] Typos in comments

## Lines of code

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L214

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L246
