# QA Report for Juicebox Buyback Delegate contest
## Overview
During the audit, 1 low and 3 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Missing check for zero address | Low | 4
NC-1 | Unused imports | Non-Critical | 1
NC-2 | Empty function bodies | Non-Critical | 1
NC-3 | Constant is missing a leading underscore | Non-Critical | 1

## Low Risk Findings(1)
### L-1. Missing check for zero address
##### Description
If address(0x0) is set it may cause the contract to revert or work wrong.
##### Instances
- [```projectToken = _projectToken;```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L124)
- [```pool = _pool;```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L125) 
- [```_projectTokenIsZero = address(_projectToken) < address(_weth);```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L127) 
- [```weth = _weth;```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L128) 

##### Recommendation
Add checks.
#
## Non-Critical Risk Findings(3)
### NC-1. Unused imports
##### Instances
- [```import "@openzeppelin/contracts/access/Ownable.sol";```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L15)

##### Recommendation
Remove the import if it is not needed.
#
### NC-2. Empty function bodies
##### Description
The code should be refactored such that empty blocks no longer exist, or the block should do something useful, such as emitting an event or reverting. Or, there should be comments why the block is empty.
##### Instances
- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

#
### NC-3. Constant is missing a leading underscore
##### Description
Internal and private constants should have a leading underscore.
##### Instances
- [```uint256 private constant SLIPPAGE_DENOMINATOR = 10000;```](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68) 

##### Recommendation
Add leading underscores for private constants.