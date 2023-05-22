## Typo mistakes
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L32

```diff
- *         of project tokens between minting using the project weigh and swapping in a
+ *         of project tokens between minting using the project weight and swapping in a
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L246

```diff
-     * @notice Swap the terminal token to receive the project toke_beforeTransferTon
+     * @notice Swap the terminal token to receive the project token_beforeTransferToken
```
## Unused imports
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol

```solidity
4: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBController3_1.sol";

8: import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleBallot.sol";
```

```
