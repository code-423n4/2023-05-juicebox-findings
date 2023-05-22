# Report
## Low Risk ##
### [L-1]: Missing checks for address(0x0)
**Context:**

1. ```projectToken = _projectToken;``` [L124](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L124)
1. ```pool = _pool;``` [L125](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L125) 
1. ```_projectTokenIsZero = address(_projectToken) < address(_weth);``` [L127](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L127) 
1. ```weth = _weth;``` [L128](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L128) 

**Recommendation:**

Add non-zero address checks when set address state variables.

## Non-Critical Issues ##
### [N-1]: Empty blocks should be removed or Emit something
**Context:**

1. ```function redeemParams(JBRedeemParamsData calldata _data)``` [L235](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235)

**Description:**

Code contains empty block

### [N-2]: Missing leading underscores in constant name
**Context:**

1. ```uint256 private constant SLIPPAGE_DENOMINATOR = 10000;``` [L68](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68)

**Description:**

Internal and private functions, state variables, constants, and immutables should starting with an underscore.

### [N-3]: Unused imports
**Context:**

```import "@openzeppelin/contracts/access/Ownable.sol";``` [L15](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L15) 
