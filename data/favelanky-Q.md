## [N-1] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: JBXBuybackDelegate.sol

235:     function redeemParams(JBRedeemParamsData calldata _data)
236:         external
237:         override
238:         returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
239:     {}

```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

</details>

## [N-2] Contract inherits `Ownable` but not use it

No function uses `onlyOwner` modifier from `Ownable` contract. If it concidered to be used in child contracts, child can inherit `Ownable` contract itself.

<details>

<summary><i>There is 1 instances of this issue:</i></summary>

```solidity

39:     contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {

```
  
https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L39

</details>

## [N-3] Event name must be in camelCase

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: JBXBuybackDelegate.sol

52:  event JBXBuybackDelegate_Swap(uint256 projectId, uint256 amountEth, uint256 amountOut);

53:  event JBXBuybackDelegate_Mint(uint256 projectId);

```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L52

</details>