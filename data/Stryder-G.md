# [G-01] Store constants in state variables instead of calculating them every time inside functions. 

Consider storing the constant numbers in state variables instead of calculating them every time inside functions. This increases
gas cost by a very little margin while deploying the contract but it saves gas in the long run 
In this case , 10 ** 18 can be stored in a state variable 

Link : https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L150

# [G-02] abi.encode is less efficient than abi.encodePacked 

Consider using abi.encodePacked() instead of abe.encode()

Link : https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L268