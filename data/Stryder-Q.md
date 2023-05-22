[NC-01] Weight Misspelt in notice tag 

The spelling of Weight has been misspelled, this degrades the quality of the documentation 

Link : https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L32

# [NC-02]  Floating Pragma Version Used

Contracts should be deployed with the same pragma version, outdated version might introduce bugs that affect the contract system negatively 
Only use floating pragma when you are using contract libraries or any other code that is meant for the consumption of developers 

Link : https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2


#[NC-03] Precision Loss can be minimized by doing all the operations before actually dividing 

Consider changing the if statement from  

    if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR))

to 

    if (_tokenCount < (_quote*(SLIPPAGE_DENOMINATOR -  _slippage)) / SLIPPAGE_DENOMINATOR)

Link: https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156

# [NC-04] Add events to functions that make important changes and transfers 

Consider adding events to the functions that make important changes to the state of the contract as well as do some 
important transfers and swaps between the tokens 

Link : https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L183
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144