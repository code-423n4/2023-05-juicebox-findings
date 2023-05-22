# QA


## L-01 Ensure 18 decimals

According to [juicebox glossary](https://docs.juicebox.money/dev/learn/glossary/tokens/)
> Projects can issue their own ERC-20 token directly from the protocol to use as its token. Projects can also bring their own token as long as it conforms to the IJBToken interface and uses **18 decimal** fixed point accounting. This makes it possible to use ERC-1155's or custom tokens.

**Recommendation**

Add check to ensure `_projectToken` has 18 decimals.

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..5743e05 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -121,6 +121,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         IUniswapV3Pool _pool,
         IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
     ) {
+        require(IJBToken(address(_projectToken)).decimals() == 18, "JBXBuybackDelegate: project token decimals must be 18");
         projectToken = _projectToken;
         pool = _pool;
         jbxTerminal = _jbxTerminal;
```

## L-02 Add check to ensure that `projectId == _data.projectId` on `payParams`

According to [juicebox glossary](https://docs.juicebox.money/dev/learn/glossary/tokens/)
> Projects can issue their own ERC-20 token directly from the protocol to use as its token. Projects can also bring their own token as long as it conforms to the IJBToken interface and uses 18 decimal fixed point accounting. This makes it possible to use ERC-1155's or custom tokens.

Note that in test you are using `Tickets` on `0x3abf2a4f8452ccc2cf7b4c1e4663147600646f66` this token doesnt respect the `IJBToken`

I think `payParams` shold implement this check `require(projectId == _data.projectId, "projectId mismatch");`

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..0a20e3f 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -78,6 +78,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     IERC20 public immutable projectToken;
 
+    /// @notice The project id
+    uint256 private immutable _projectId;
+
     /**
      * @notice The uniswap pool corresponding to the project token-other token market
      *         (this should be carefully chosen liquidity wise)
@@ -122,6 +125,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
     ) {
         projectToken = _projectToken;
+        projectId = JBXToken(address(_projectToken)).projectId();
         pool = _pool;
         jbxTerminal = _jbxTerminal;
         _projectTokenIsZero = address(_projectToken) < address(_weth);
@@ -146,6 +150,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         override
         returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
     {
+        require(projectId == _data.projectId, "projectId mismatch");
         // Find the total number of tokens to mint, as a fixed point number with 18 decimals
         uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
```

## L-03 Since `projectToken` can be any token custom token its possible to reenter

`projectToken` could be a token with hooks, opening the possibility of reentrancy from `_data.beneficiary` after the token is transfered to `_data.beneficiary` in the `_swap` function on [JBXBuybackDelegate.sol#L286](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L286)

## L-04 Since `projectToken` can be any token custom token its possible to dos

`projectToken` could be a token with hooks, opening the possibility of DOS. The attacker in this case is the `_data.beneficiary` and he just have to craft a contract that consume all the gas on the hook on transfer triggered on  [JBXBuybackDelegate.sol#L286](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L286)



## NC Use `IERC20` instead of `IJBToken`

According to [juicebox glossary](https://docs.juicebox.money/dev/learn/glossary/tokens/)
> Projects can issue their own ERC-20 token directly from the protocol to use as its token. Projects can also bring their own token as long as it conforms to the IJBToken interface and uses 18 decimal fixed point accounting. This makes it possible to use ERC-1155's or custom tokens.

Therefore i infer that `IERC20 _projectToken` is a `IJBToken` and would be clear if you just use `IJBToken` and remove `IERC20`

**Recommendation**
Out of scope but important, `interface IJBToken` should extend the `IERC20`; `interface IJBToken is IERC20`

<details>
<summary>Diff</summary>

```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..ab53cd7 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -13,7 +13,6 @@ import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBDidPayData.sol";
 import "@jbx-protocol/juice-contracts-v3/contracts/structs/JBPayParamsData.sol";
 
 import "@openzeppelin/contracts/access/Ownable.sol";
-import "@openzeppelin/contracts/interfaces/IERC20.sol";
 
 import "@paulrberg/contracts/math/PRBMath.sol";
 
@@ -76,7 +75,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      * 
      * @dev In this context, this is the tokenOut
      */
-    IERC20 public immutable projectToken;
+    IJBToken public immutable projectToken;
 
     /**
      * @notice The uniswap pool corresponding to the project token-other token market
@@ -116,7 +115,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      * @dev No other logic besides initializing the immutables
      */
     constructor(
-        IERC20 _projectToken,
+        IJBToken _projectToken,
         IWETH9 _weth,
         IUniswapV3Pool _pool,
         IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
```

</details>


## NC Remove unused imports

`@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleBallot.sol` is not being use and can be remove

## NC Use named imports

Instead of `import "file.sol";` use `import {File} from "file.sol"`, example;
```solidity
import {IJBController} from "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBController3_1.sol";
```

## NC Use fixed pragma

In [JBXBuybackDelegate.sol#L2](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L2)
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

Take a look into [SWC-103](https://swcregistry.io/docs/SWC-103)


## NC Ownable is not used and should be remove

There is no function using the `onlyOwner` modifier from `Ownable` imported in [JBXBuybackDelegate.sol#L15](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L15) i think this import should be removed.


## NC Better vatiable naming

- Variable `SLIPPAGE_DENOMINATOR` should be renamed to `BPS`, [JBXBuybackDelegate.sol#L68](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L68)
- Variable `mintedAmount` should be renamed `mutexMintedAmount`, [JBXBuybackDelegate.sol#L106](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L106)
- Variable `reservedRate` should be renamed `mutexReservedRate`, [JBXBuybackDelegate.sol#L113](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L113)

## NC Private variables dont respect naming convention

All private variables should start with `_` but [`SLIPPAGE_DENOMINATOR`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L68), [`mintedAmount`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L106) and [JBXBuybackDelegate.sol#L113](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L113) dont respect the convetion.

## NC Immutables variables dont respect naming convention

Immutables should be in uppercase like [`SLIPPAGE_DENOMINATOR`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L68), but 
[`_projectTokenIsZero`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#63), [`projectToken`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L79) and [`pool`](https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L85) dont respect the convention

## Typo in comment

`* @notice Mint the token out, sending back the token in in the terminal` on should be;
```diff
diff --git a/juice-buyback/contracts/JBXBuybackDelegate.sol b/juice-buyback/contracts/JBXBuybackDelegate.sol
index 0ee751b..5a59e9a 100644
--- a/juice-buyback/contracts/JBXBuybackDelegate.sol
+++ b/juice-buyback/contracts/JBXBuybackDelegate.sol
@@ -326,7 +326,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
     }
 
     /**
-     * @notice Mint the token out, sending back the token in in the terminal
+     * @notice Mint the token out, sending back the token in the terminal
      *
      * @param  _data the didPayData passed by the terminal
      * @param  _amount the amount of token out to mint
```