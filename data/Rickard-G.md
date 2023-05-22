# [G-01] Open the optimizer
## Lines of code
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/foundry.toml#L1-L7](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/foundry.toml#L1-L7)
## Vulnerability details
### Impact
Always use the Solidity optimizer to optimize gas costs. It’s good practice to set the optimizer as high as possible until it no longer helps reduce gas costs in function calls. This is advisable since function calls are intended to be executed many more times than contract deployment, which only happens once.
## Recommended mitigation steps
In the light of this information, I suggest you to open the optimizer for `juice-buyback`.
# [G-02] Use a more recent version of solidity
## Lines of code
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2)
## Vulnerability details
### Impact
- Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value
- In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
- In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.       
       
1 result - 1 file
````solidity
juice-buyback/contracts/JBXBuybackDelegate.sol

2:  pragma solidity ^0.8.16;
````
[https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2)
## Tools Used
Manual Review
## Recommended mitigation steps
````diff
- 2:  pragma solidity ^0.8.16;
+ 2:  pragma solidity 0.8.17;
````