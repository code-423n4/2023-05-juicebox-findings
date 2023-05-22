# [L-01]: Unnecessary computation when `_data.preferClaimedTokens=false`

### Code snippet
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L195-L206

Line number from `195-197` always getting executed, even when those variables are not needed in subsequent call when `_data.preferClaimedTokens=false`.
If line number `195-197` moved inside if block at line 200, we can save some gas for the call, by avoiding computation when `_data.preferClaimedTokens = false`

```diff
diff --git a/JBXBuybackDelegate_orig.sol b/JBXBuybackDelegate_mod.sol
index 0ee751b..44717ec 100644
--- a/JBXBuybackDelegate_orig.sol
+++ b/JBXBuybackDelegate_mod.sol
@@ -192,12 +192,12 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         uint256 _reservedRate = reservedRate;
         reservedRate = 1;
 
-        // The minimum amount of token received if swapping
-        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
-        uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
-
         // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
         if (_data.preferClaimedTokens) {
+            // The minimum amount of token received if swapping
+            (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
+            uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
+
             // Try swapping
             uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);
```

# [L-02] No sanity checks in the constructor can lead to issues or redeployments
## Code snippet
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129
## Vulnerability description
In the constructor no checks are done for the addresses the contract is setting (like the project token, the pool...). Even if the deployer is trusted, it is normal to have typos or man-made mistakes and those mistakes here could mean a bricked contract or the need to redeploy it.

The constructor should perform some validations to the addresses sent or at least, check they are not address zero.
## Mitigation
Include some argument validation in the constructor.


# [L-03] Inconsistency between documentation and code around ERC20 compliance

### Code snippet
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L119

### Description

In the [Juicebox documentation](https://docs.juicebox.money/dev/learn/glossary/tokens/) it states:

> By default, the protocol allocates tokens to recipients using an internal accounting mechanism in [JBTokenStore](https://docs.juicebox.money/dev/api/contracts/jbtokenstore/). These are fungible but do not conform to the ERC-20 standard, and as such cannot be composed with ecosystem ERC-20/ERC-721 marketplaces like AMMs and Opensea. Their balances can be used for voting on various platforms.
> Projects can issue their own ERC-20 token directly from the protocol to use as its token. Projects can also bring their own token as long as it conforms to the [IJBToken](https://docs.juicebox.money/dev/api/interfaces/ijbtoken/) interface and uses 18 decimal fixed point accounting. This makes it possible to use ERC-1155's or custom tokens.

This suggest that the tokens used by the project do not need to conform to ERC20 standard. In JBXBuybackDelegate however `projectToken` is casted as `IERC20`:

```
constructor(
    IERC20 _projectToken,
```

# [L-04] Address of tokens in pool can be different than the ones passed to the constructor.

### Code snippet
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129

### Vulnerability description
In constructor, there is no implementation to ensure that the `projectToken` and `weth` addresses are the same as the addresses of the tokens swapped in `pool`. This can lead lack of synchronisation between pool and tokens, forcing the contract to be redeployed.

### Mitigation
Get token addresses from a trusted source or verify token addresses passed in constructor against the ones stored in a pool contract.

# [L-05] Unnecessary calculation of `_nonReservedTokenInContract`:
## Code Snippet
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312

## Description
The following line is unnecesary to calculate because it's already been calculated before so we can get the older variable.
```solidity
uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
```
Always `_nonReservedTokenInContract` will be equal to `_nonReservedToken`, calculated here:
```solidity
// The amount to send to the beneficiary
uint256 _nonReservedToken = PRBMath.mulDiv(
    _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
);

// The amount to add to the reserved token
uint256 _reservedToken = _amountReceived - _nonReservedToken;
```