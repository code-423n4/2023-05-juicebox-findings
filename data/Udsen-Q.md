## 1. PROPER `natspec` COMMENTING STANDARD IS NOT USED.

Proper Natspec commenting standard is not used and the comments are not clear for the developers and auditors to understand.

Observed across entire `JBXBuybackDelegate` smart contract.

## 2. `Typos` DETECTED IN THE `natspec` COMMENTS.

     * @notice The Uniswap V3 pool callback (where token transfer should `happens`)
	 
Should be corrected as `happen`	 
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L212

     * @dev    Slippage `controle` is achieved here
	 
Should be corrected as `control`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L214

     * @notice Mint the token out, sending back the token in `in` the terminal 
	 
An extra `in` should be removed from the comment.
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L329

     * @notice Swap the terminal token to receive the project `toke_beforeTransferTon`
	 
Typo in the above comment in the highlighted section.
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L246

    // 2) Mint the `reserved token` with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
	
Above comment should be corrected as below for better understanding.
	
    // 2) Mint the `_amountReceived` with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L301