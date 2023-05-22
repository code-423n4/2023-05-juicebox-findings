
# Low Risk and Non-Critical Issues

# [L-01] Missing checks for address(0x0) when assigning values to address state variables 

Zero-address check should be used in function, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L124


# [L-02] Missing initializer modifier on constructor

OpenZeppelin recommends that the initializer modifier be applied to constructors in order to avoid potential griefs, social engineering, or exploits. 
Link to the code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118



# [N-01] Pragma Floating
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2



# [N-02] Use a modifier for access control (03 Instances)

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

Link to the code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L185
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L218

# [N-03] Empty blocks should be removed (01 Instances)

Avoid using code blocks or use them for emitting events.

Link to the code:

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L239
