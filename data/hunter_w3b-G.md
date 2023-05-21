# Gas Optimization

# Summary

| Number | Optimization Details                                                                                                     | Context |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | State variables can be cached instead of `re-reading` them from `storage`                                                |    7    |
| [G-02] | Avoid `contract` existence checks by using low level calls                                                               |    7    |
| [G-03] | Tightly pack `storage` variables/optimize the order of variable declaration                                              |    3    |
| [G-04] | `_amountToSend` should be checked for 0 before calling a transfer                                                        |    1    |
| [G-05] | Add Solidity's `optimizer`                                                                                               |    1    |
| [G-06] | Use hardcode address instead `address(this)`                                                                             |    4    |
| [G-07] | Empty blocks should be `removed` or `emit` something                                                                     |    1    |
| [G-08] | Using `> 0` costs less gas than `!= 0` when used on a unsigned integers                                                  |    3    |
| [G-09] | `abi.encode()` is less efficient than `abi.encodePacked()`                                                               |    1    |
| [G-10] | Not using the named `return variables` when a function returns, wastes deployment gas                                    |    4    |
| [G-11] | Use solidity version `0.8.19` to gain some gas boost                                                                     |    1    |
| [G-12] | Failure to check the zero address in the `JBXBuybackDelegate.sol.constructor()` causes the contract to be deployed again |    1    |
| [G-13] | `bool` variables on storage usage and gas consumption                                                                    |    1    |

## [G-01] State variables can be cached instead of `re-reading` them from `storage`

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

Total Instances: `7`

Gas savings: `7 * 100 = 700`

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224

### Loading `_projectTokenIsZero` into a local variable slightly reduced bytecode size and more gas efficient save `2 SLOAD`

```solidity
File: contracts/JBXBuybackDelegate.sol

224:         uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
225:         uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
```

```diff
diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..f0f47f6 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -221,8 +221,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

         // Assign 0 and 1 accordingly
-        uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
-        uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
+        bool projectTokenIsZero = _projectTokenIsZero;
+        uint256 _amountReceived = uint256(-(projectTokenIsZero ? amount0Delta : amount1Delta));
+        uint256 _amountToSend = uint256(projectTokenIsZero ? amount1Delta : amount0Delta);

         // Revert if slippage is too high
         if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L275

### Loading `_projectTokenIsZero` into a local variable slightly reduced bytecode size and more gas efficient save `2 SLOAD`

```solidity
File: contracts/JBXBuybackDelegate.sol

258:   function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
259:        internal
260:        returns (uint256 _amountReceived)
261:    {
262:        // Pass the token and min amount to receive as extra data
263:       try pool.swap({
264:            recipient: address(this),
265:            zeroForOne: !_projectTokenIsZero,
266:            amountSpecified: int256(_data.amount.value),
267:            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
268:            data: abi.encode(_minimumReceivedFromSwap)
269:        }) returns (int256 amount0, int256 amount1) {
270:            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
271:           _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
272:       } catch {
273:            // implies _amountReceived = 0 -> will later mint when back in didPay
274:           return _amountReceived;
275:        }
```

```diff
diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..f82e409 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -260,15 +260,16 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         returns (uint256 _amountReceived)
     {
         // Pass the token and min amount to receive as extra data
+           bool projectTokenIsZero = _projectTokenIsZero;
         try pool.swap({
             recipient: address(this),
             zeroForOne: !_projectTokenIsZero,
             amountSpecified: int256(_data.amount.value),
-            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
+            sqrtPriceLimitX96: projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
             data: abi.encode(_minimumReceivedFromSwap)
         }) returns (int256 amount0, int256 amount1) {
             // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
-            _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
+            _amountReceived = uint256(-(projectTokenIsZero ? amount0 : amount1));
         } catch {
             // implies _amountReceived = 0 -> will later mint when back in didPay
             return _amountReceived;

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L156

### Loading `SLIPPAGE_DENOMINATOR` into a local variable slightly reduced bytecode size and more gas efficient save `1 SLOAD`

```solidity
File: contracts/JBXBuybackDelegate.sol

156:        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
157:            // Pass the quote and reserve rate via a mutex
158:            mintedAmount = _tokenCount;
159:            reservedRate = _data.reservedRate;
160:
161:            // Return this delegate as the one to use, and do not mint from the terminal
162:            delegateAllocations = new JBPayDelegateAllocation[](1);
163:            delegateAllocations[0] =
164:                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});
165:
166:            return (0, _data.memo, delegateAllocations);
167:        }
```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..aa7fa0b 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -153,7 +153,8 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

         // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
-        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
+        uint256  _SLIPPAGE_DENOMINATOR = SLIPPAGE_DENOMINATOR;
+        if (_tokenCount < _quote - (_quote * _slippage / _SLIPPAGE_DENOMINATOR)) {
             // Pass the quote and reserve rate via a mutex
             mintedAmount = _tokenCount;
             reservedRate = _data.reservedRate;

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L218

### Loading `pool` into a local variable slightly reduced bytecode size and more gas efficient save `1 SLOAD`

```solidity
File: contracts/JBXBuybackDelegate.sol

216:    function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
217:        // Check if this is really a callback
218:        if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
219:
220:        // Unpack the data
221:        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..5fe5508 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -215,7 +215,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
         // Check if this is really a callback
-        if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
+
+        IUniswapV3Pool  _pool  =  pool;
+        if (msg.sender != address(_pool)) revert JuiceBuyback_Unauthorized();

         // Unpack the data
         (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L185

### Loading `jbxTerminal` into a local variable slightly reduced bytecode size and more gas efficient save `1 SLOAD`

```solidity
File: contracts/JBXBuybackDelegate.sol

183:    function didPay(JBDidPayData calldata _data) external payable override {
184:        // Access control as minting is authorized to this delegate
185:        if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
186:
187:        // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
188:        uint256 _tokenCount = mintedAmount;
189:        mintedAmount = 1;
```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..cb4dea6 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -182,7 +182,8 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     function didPay(JBDidPayData calldata _data) external payable override {
         // Access control as minting is authorized to this delegate
-        if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
+       IJBPayoutRedemptionPaymentTerminal3_1  _jbxTerminal  =  jbxTerminal;
+        if (msg.sender != address(_jbxTerminal)) revert JuiceBuyback_Unauthorized();

         // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
         uint256 _tokenCount = mintedAmount;
```

## [G-02] Avoid `contract` existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Total Instances: `7`

Gas savings: `7 * 100 = 700`

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol

```solidity
File: contracts/JBXBuybackDelegate.sol

/// @audit mulDiv()
150:         uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);


/// @audit deposit()
231:         weth.deposit{value: _amountToSend}();


/// @audit transfer()
232:        weth.transfer(address(pool), _amountToSend);



/// @audit swap()
263:         try pool.swap({


/// @audit mulDiv()
278:         uint256 _nonReservedToken = PRBMath.mulDiv(


/// @audit transfer()
286:         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);


/// @audit addToBalanceof()
348:        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
```

## [G-03] Tightly pack `storage` variables/optimize the order of variable declaration

Here, the storage variables can be tightly packed by putting data type that can fit together next to each other.

Three state varaible packaging in one storage slot.

State variable packaging in JBXBuybackDelegate.sol contract `(2 * 2k = 4k gas saved)`

```solidity
File: contracts/JBXBuybackDelegate.sol

65     /**
66     * @notice The unit of the max slippage (expressed in 1/10000th)
67     */
68    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
69
70    //*********************************************************************//
71    // --------------------- public constant properties ------------------ //
72    //*********************************************************************//
73
74    /**
75     * @notice The project token address
76     *
77     * @dev In this context, this is the tokenOut
78     */
79    IERC20 public immutable projectToken;
80
81    /**
82     * @notice The uniswap pool corresponding to the project token-other token market
83     *         (this should be carefully chosen liquidity wise)
84     */
85    IUniswapV3Pool public immutable pool;
86
87    /**
88     * @notice The project terminal using this extension
89     */
90    IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;
91
92   /**
93     * @notice The WETH contract
94     */
95    IWETH9 public immutable weth;
96
97    //*********************************************************************//
98    // --------------------- private stored properties ------------------- //
99    //*********************************************************************//
100
101    /**
102     * @notice The amount of token created if minted is prefered
103     *
104     * @dev    This is a mutex 1-x-1
105     */
106    uint256 private mintedAmount = 1;
107
108    /**
109     * @notice The current reserved rate
110     *
111     * @dev    This is a mutex 1-x-1
112     */
113    uint256 private reservedRate = 1;
114
115    /**
116     * @dev No other logic besides initializing the immutables
117     */

```

```diff
diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..222def9 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -65,7 +65,29 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
     /**
      * @notice The unit of the max slippage (expressed in 1/10000th)
      */
-    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
+    uint64 private constant SLIPPAGE_DENOMINATOR = 10000;
+
+  //*********************************************************************//
+    // --------------------- private stored properties ------------------- //
+    //*********************************************************************//
+
+    /**
+     * @notice The amount of token created if minted is prefered
+     *
+     * @dev    This is a mutex 1-x-1
+     */
+    uint64 private mintedAmount = 1;
+
+    /**
+     * @notice The current reserved rate
+     *
+     * @dev    This is a mutex 1-x-1
+     */
+    uint64 private reservedRate = 1;
+
+    /**
+     * @dev No other logic besides initializing the immutables
+     */

     //*********************************************************************//
     // --------------------- public constant properties ------------------ //
@@ -94,27 +116,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     IWETH9 public immutable weth;

-    //*********************************************************************//
-    // --------------------- private stored properties ------------------- //
-    //*********************************************************************//
-
-    /**
-     * @notice The amount of token created if minted is prefered
-     *
-     * @dev    This is a mutex 1-x-1
-     */
-    uint256 private mintedAmount = 1;
-
-    /**
-     * @notice The current reserved rate
-     *
-     * @dev    This is a mutex 1-x-1
-     */
-    uint256 private reservedRate = 1;
-
-    /**
-     * @dev No other logic besides initializing the immutables
-     */
+
     constructor(
         IERC20 _projectToken,
         IWETH9 _weth,
(END)

```

## [G-04] `_amountToSend` should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L232

```solidity
File: contracts/JBXBuybackDelegate.sol

232:        weth.transfer(address(pool), _amountToSend);

```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..54ee873 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -229,7 +229,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw

         // Wrap and transfer the weth to the pool
         weth.deposit{value: _amountToSend}();
-        weth.transfer(address(pool), _amountToSend);
+        if (_amountToSend != 0)  weth.transfer(address(pool), _amountToSend);
     }

     function redeemParams(JBRedeemParamsData calldata _data)

```

## [G-05] Add Solidity's `optimizer`

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

```javascript
File: juice-buyback/foundry.toml


[profile.default]
src = 'contracts'
out = 'out'
libs = ['lib', 'node_modules']
fs_permissions = [{ access = "read", path = "./node_modules/@jbx-protocol/juice-contracts-v3/deployments/mainnet"}] # Get the deployment addresses for forking

# See more config options https://github.com/foundry-rs/foundry/tree/master/config
```

```javascript
File: juice-buyback/foundry.toml

[profile.default]
src = 'contracts'
out = 'out'
libs = ['lib', 'node_modules']
fs_permissions = [{ access = "read", path = "./node_modules/@jbx-protocol/juice-contracts-v3/deployments/mainnet"}] # Get the deployment addresses for forking

+   optimizer = true
+   optimizer_runs = 200

# See more config options https://github.com/foundry-rs/foundry/tree/master/config
```

## [G-06] Use hardcode address instead `address(this)`

Instead of `address(this)`, it is more gas-efficient to pre-calculate and use the address with a hardcode.

Foundry's script.sol and solmate```LibRlp.sol` contracts can do this.

https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L264

```solidity
File: contracts/JBXBuybackDelegate.sol


264:             recipient: address(this),


294:                 _holder: address(this),


305:                 _beneficiary: address(this),


316:                     _holder: address(this),
```

## [G-07] Empty blocks should be `removed` or `emit` something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

```solidity
File: contracts/JBXBuybackDelegate.sol


235:    function redeemParams(JBRedeemParamsData calldata _data)
236:        external
237:        override
238:        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
239:    {}

```

## [G-08] Using `> 0` costs less gas than `!= 0` when used on a unsigned integers

This change saves 3 gas per instance

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L286

```solidity
File:  contracts/JBXBuybackDelegate.sol


286:        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);


289:         if (_reservedToken != 0) {


314:            if (_nonReservedTokenInContract != 0) {
```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..e9f1b41 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -283,10 +283,10 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         uint256 _reservedToken = _amountReceived - _nonReservedToken;

         // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
-        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
+        if (_nonReservedToken  >  0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

         // If there are reserved token, add them to the reserve
-        if (_reservedToken != 0) {
+        if (_reservedToken  >  0) {
             IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

             // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
@@ -311,7 +311,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
             uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

-            if (_nonReservedTokenInContract != 0) {
+            if (_nonReservedTokenInContract > 0) {
                 controller.burnTokensOf({
                     _holder: address(this),
                     _projectId: _data.projectId,


```

## [G-09] `abi.encode()` is less efficient than `abi.encodePacked()`

Gas savings: 100

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L268

```solidity
File: contracts/JBXBuybackDelegate.sol

268:             data: abi.encode(_minimumReceivedFromSwap)

```

```diff
diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..335dc74 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -265,7 +265,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             zeroForOne: !_projectTokenIsZero,
             amountSpecified: int256(_data.amount.value),
             sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
-            data: abi.encode(_minimumReceivedFromSwap)
+            data: abi.encodePacked(_minimumReceivedFromSwap)
         }) returns (int256 amount0, int256 amount1) {
             // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
             _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));

```

## [G-10] Not using the named `return variables` when a function returns, wastes deployment gas

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L147

```solidity
File: contracts/JBXBuybackDelegate.sol

147:        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)


238:        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)


260:        returns (uint256 _amountReceived)

269:        }) returns (int256 amount0, int256 amount1) {
```

```diff

diff --git a/JBXBuybackDelegate.sol.origi b/JBXBuybackDelegate.sol
index 0ee751b..eb4f8a4 100644
--- a/JBXBuybackDelegate.sol.origi
+++ b/JBXBuybackDelegate.sol
@@ -144,7 +144,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
     function payParams(JBPayParamsData calldata _data)
         external
         override
-        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
+        returns (uint256  , string memory , JBPayDelegateAllocation[] memory  )
     {
         // Find the total number of tokens to mint, as a fixed point number with 18 decimals
         uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
@@ -235,7 +235,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
     function redeemParams(JBRedeemParamsData calldata _data)
         external
         override
-        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
+        returns (uint256 , string memory , JBRedemptionDelegateAllocation[] memory )
     {}

     //*********************************************************************//
@@ -257,7 +257,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
         internal
-        returns (uint256 _amountReceived)
+        returns (uint256 )
     {
         // Pass the token and min amount to receive as extra data
         try pool.swap({
@@ -266,7 +266,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             amountSpecified: int256(_data.amount.value),
             sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
             data: abi.encode(_minimumReceivedFromSwap)
-        }) returns (int256 amount0, int256 amount1) {
+        }) returns (int256  , int256  ) {
             // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
             _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
         } catch {

```

## [G-11] Use solidity version `0.8.19` to gain some gas boost

Upgrade to the solidity version 0.8.19 to get additional gas savings.

This is a good reference that why you should upgrade to 0.8.19 to savings gas:
https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2

```solidity
2:  pragma solidity ^0.8.16;

```

## [G-12] Failure to check the zero address in the `JBXBuybackDelegate.sol.constructor()` causes the contract to be deployed again

Zero address control is not performed in the constructor in contract within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129

```solidity
File: contracts/JBXBuybackDelegate.sol


118    constructor(
119        IERC20 _projectToken,
120        IWETH9 _weth,
121        IUniswapV3Pool _pool,
122        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
123    ) {
124        projectToken = _projectToken;
125        pool = _pool;
126        jbxTerminal = _jbxTerminal;
127        _projectTokenIsZero = address(_projectToken) < address(_weth);
128        weth = _weth;
129    }

```

## [G-13] `bool` variables on storage usage and gas consumption

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

below is good example you can check:
https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63

```solidity
File: contracts/JBXBuybackDelegate.sol


63:     bool private immutable _projectTokenIsZero;

```
