### ABI decoding the quote in `didPay()` can be done inside the following if block

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L196

Line 196 and 197 can be put inside the following if block to reduce gas cost when `preferClaimedTokens = false`

```diff
-   (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
-   uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

if (_data.preferClaimedTokens) {

+   (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
+   uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);


   // Try swapping
   uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, _reservedRate);

   // If swap failed, mint instead, with the original weight + add to balance the token in
   if (_amountReceived == 0) _mint(_data, _tokenCount);
}
```

### Could do only 1 burn in `_swap` and remove `_nonReservedTokenInContract` variable

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L293

The first burn could be removed and still burn all the tokens needed by changing the amount burned in the second burn (line 315) and setting `_preferClaimedTokens: true`

The new value of `_tokenCount` in the burn should be `_amountReceived` because the contract still has the reserveToken from the swap and then get the nonReserveToken from the mint which equals to `_amountReceived`

```diff

- controller.burnTokensOf({
-    _holder: address(this),
-    _projectId: _data.projectId,
-    _tokenCount: _reservedToken,
-    _memo: "",
-    _preferClaimedTokens: true
- })

...

- uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

- controller.burnTokensOf({
-    _holder: address(this),
-    _projectId: _data.projectId,
-    _tokenCount: _nonReservedTokenInContract,
-    _memo: "",
-    _preferClaimedTokens: false
- });

+ controller.burnTokensOf({
+    _holder: address(this),
+    _projectId: _data.projectId,
+    _tokenCount: _amountReceived,
+    _memo: "",
+    _preferClaimedTokens: true
+ });
```

### `slippage` variable is not needed

The `slippage` variable is used to determine `_minimumReceivedFromSwap` from the swap but this could be calculated offchain so the `quote` variable could directly be the minimum returns wanted from the swap.

This would save gas in `payParams()` and `didPay()`.

To make it more clear, quote should probably be renamed `_minimumReceivedFromSwap`.

In `payParams()`:
```diff
- // Unpack the quote from the pool, given by the frontend
- (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

- // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
- if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {

+ // Unpack the _minimumReceivedFromSwap from the pool, given by the frontend
+ (,, uint256 _minimumReceivedFromSwap) = abi.decode(_data.metadata, (bytes32, bytes32, uint256));

+ // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
+ if (_tokenCount < _minimumReceivedFromSwap) {
```

In `didPay()`:
```diff
- (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
- uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

+ (,, uint256 _minimumReceivedFromSwap) = abi.decode(_data.metadata, (bytes32, bytes32, uint256));
```
