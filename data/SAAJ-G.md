 
# Gas Optimizations Report

This report focuses on Juice-buyback Protocol contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency for the codebase in-scope of Juice-buyback Protocol are categorised into 08 main areas; with further multiple instances in each of the category.

# [G-01] Use solidity version 0.8.19 to gain some gas boost (01 Instances)

Use latest version of solidity i.e., 0.8.19 for getting additional gas saving for reference check this article.

Link to the code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol


# [G-02] Immutable has more gas efficiency than constant (01 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68


# [G-03] Use hardcode address instead address(this) (04 Instances)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

Link to the code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L264
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L294
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L305
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L316


# [G 04] Using bools for storage incurs overhead (02 Instances)

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back.
This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
	
Link to the Code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L127


# [G 05] Declare Public library as internal library (01 Instances)

External call to a public library function is very expensive for reference checks this article. When library has only internal functions it can be used as internal library.

Link to the code:
1.	https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL22C8-L22C60



# [G-06] Use uint256(1)/uint256(2) instead for true and false boolean states (05 Instances)

Boolean for storage if not used, saves Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

Link to the Code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L298
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L308
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L307
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L320
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L344


# [G 07] abi.encode() is less efficient than abi.encodepacked()(01 Instances)

Refer to this article.

Link to the Code:
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L268

# [G 08] String literals passed to abi.encode()/abi.decode() should not be split by commas (03 Instances)

String literals can be split into multiple parts and still be considered as a single string literal.
EACH new comma costs 21 gas due to stack operations and separate MSTOREs.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L153
2.	https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L196
3.	https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L221

