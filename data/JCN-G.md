# Summary
All of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.19`. Some optimizations are also explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions (with all the optimizations applied):

| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| JBXBuybackDelegate.didPay |  242106  |  240737 |  1369 | 
| JBXBuybackDelegate.payParams |  8271  |  5393 |  2878 | 
| JBXBuybackDelegate.uniswapV3SwapCallback |  20125  |  19772 | 353 | 

**Total gas saved across all listed functions: 4600**

*Notes*: 

- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diff for the contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/28ddc1adcacdee9691ce126d23d7a1d6).
- The `Avg` gas for `didPay` changes between tests and so the `Max` column (which remains the same) is used when benchmarking this function.
- Only runtime gas is highlighted above, as it will inevitably outweight deployment gas costs throughout the lifetime of the protocol. 
- Substantial deployment gas savings are highlighted in their appropriate instances and are only provided as a reference.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:| 
| [G-01](#state-variables-can-be-packed-to-use-fewer-storage-slots) | State variables can be packed to use fewer storage slots | 1 | 
| [G-02](#create-immutable-variable-to-avoid-redundant-external-calls) | Create immutable variable to avoid redundant external calls | 1 | 
| [G-03](#use-assembly-to-perform-efficient-back-to-back-calls) | Use assembly to perform efficient back-to-back calls | 1 | 
| [G-04](#use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 3 | 
| [G-05](#use-assembly-to-emit-events) | Use assembly to emit events | 2 | 
| [G-06](#use-assembly-to-validate-msgsender) | Use assembly to validate `msg.sender` | 2 | 
| [G-07](#use-assembly-to-efficiently-read-from-and-write-to-packed-storage-slots) | Use assembly to efficiently read from and write to packed storage slots | 2 |

## State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

**Note: This optimization will save ~20_000 deployment gas since we are avoiding one `Gsset (20000 gas)` during deployment. However, only runtime gas is bencharked since runtime gas is paid multiple times over, while deployment gas is only paid once**.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L106-L113

*Gas Savings for `JBXBuybackDelegate.payParams`, obtained via protocol's tests: Avg 2743 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  9981    |  12476   |  8271   |    10    |
| After  |  6771    |  7523    |  5528   |    10    |

### Pack `mintedAmount` and `reservedRate` into one storage slot to save 1 SLOT (~2000 gas)
```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
106:    uint256 private mintedAmount = 1;
107:
108:    /**
109:     * @notice The current reserved rate
110:     * 
111:     * @dev    This is a mutex 1-x-1
112:     */
113:    uint256 private reservedRate = 1;
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..23e2aa2 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -103,14 +103,14 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      *
      * @dev    This is a mutex 1-x-1
      */
-    uint256 private mintedAmount = 1;
+    uint128 private mintedAmount = 1;

     /**
      * @notice The current reserved rate
      *
      * @dev    This is a mutex 1-x-1
      */
-    uint256 private reservedRate = 1;
+    uint128 private reservedRate = 1;

     /**
      * @dev No other logic besides initializing the immutables
@@ -155,8 +155,8 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
         if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
             // Pass the quote and reserve rate via a mutex
-            mintedAmount = _tokenCount;
-            reservedRate = _data.reservedRate;
+            mintedAmount = uint128(_tokenCount);
+            reservedRate = uint128(_data.reservedRate);

             // Return this delegate as the one to use, and do not mint from the terminal
             delegateAllocations = new JBPayDelegateAllocation[](1);
```

## Create immutable variable to avoid redundant external calls
In the instances below, an external call to retrieve the `directory` is performed each time `_swap` and `_mint` is invoked. We can avoid performing this call on each invocation by executing this external call once in the constructor and storing the `directory` as an immutable variable. Doing so will save 2 external calls each time `didPay` is called (`didPay` invokes `_swap` & `_mint`).

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L290

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L335

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 602 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148266  |  242106  |  161126 |    7     |
| After  |  147664  |  241504  |  160575 |    7     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
290:        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId)); // called in `_swap`

335:        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId)); // called in `_mint`
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..403216d 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -89,6 +89,8 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;

+    IJBDirectory private immutable directory;
+
     /**
      * @notice The WETH contract
      */
@@ -124,6 +126,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         projectToken = _projectToken;
         pool = _pool;
         jbxTerminal = _jbxTerminal;
+        directory = jbxTerminal.directory();
         _projectTokenIsZero = address(_projectToken) < address(_weth);
         weth = _weth;
     }
@@ -287,7 +290,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw

         // If there are reserved token, add them to the reserve
         if (_reservedToken != 0) {
-            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+            IJBController controller = IJBController(directory.controllerOf(_data.projectId));

             // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
             controller.burnTokensOf({
@@ -332,7 +335,7 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      * @param  _amount the amount of token out to mint
      */
     function _mint(JBDidPayData calldata _data, uint256 _amount) internal {
-        IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));
+        IJBController controller = IJBController(directory.controllerOf(_data.projectId));

         // Mint to the beneficiary with the fc reserve rate
         controller.mintTokensOf({
```

## Use assembly to perform efficient back-to-back calls
If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (`scratch space` + `free memory pointer`), which can potentially allow us to avoid memory expansion costs. In this case we are also able to efficiently store both function signatures together in memory as one word, saving one MLOAD in the process.

**Note: In order to do this optimization safely we will cache and restore the free memory pointer after we are done with our function calls**.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L231-L232

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 392 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148266  |  242106  |  161126 |    7     |
| After  |  147874  |  241714  |  160902 |    7     |

*Gas Savings for `JBXBuybackDelegate.uniswapV3SwapCallback`, obtained via protocol's tests: Avg 262 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28794   |  30714   |  20125  |    6     |
| After  |  28402   |  30322   |  19863  |    6     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
231:        weth.deposit{value: _amountToSend}();
232:        weth.transfer(address(pool), _amountToSend);
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..9c252d5 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -228,8 +228,20 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

         // Wrap and transfer the weth to the pool
-        weth.deposit{value: _amountToSend}();
-        weth.transfer(address(pool), _amountToSend);
+        IWETH9 _weth = weth;
+        IUniswapV3Pool _pool = pool;
+        assembly {
+            // function selectors for `deposit()` & `transfer(address,uint256)`
+            mstore(0x00, 0xd0e30db0a9059cbb)
+            if iszero(call(gas(), _weth, _amountToSend, 0x18, 0x04, 0x00, 0x00)) {revert(0, 0)}
+            // store memory pointer
+            let memptr := mload(0x40)
+            mstore(0x20, _pool)
+            mstore(0x40, _amountToSend)
+            if iszero(call(gas(), _weth, 0x00, 0x1c, 0x44, 0x00, 0x00)) {revert(0, 0)}
+            // restore memory pointer
+            mstore(0x40, memptr)
+        }
     }

     function redeemParams(JBRedeemParamsData calldata _data)
```

## Use assembly in place of `abi.decode` to extract calldata values more efficiently
Instead of using `abi.decode`, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

Total Instances: `3`

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L153

*Gas Savings for `JBXBuybackDelegate.payParams`, obtained via protocol's tests: Avg 112 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  9981    |  12476   |  8271   |    10    |
| After  |  9886    |  12357   |  8159   |    10    |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
153:        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..6319382 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -150,7 +150,15 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

         // Unpack the quote from the pool, given by the frontend
-        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
+        uint256 _quote;
+        uint256 _slippage;
+        { // used to discard `data` variable and avoid extra stack manipulation
+            bytes calldata data = _data.metadata;
+            assembly {
+                _quote := calldataload(add(data.offset, 0x40))
+                _slippage := calldataload(add(data.offset, 0x60))
+            }
+        }

         // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
         if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L196

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 95 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148266  |  242106  |  161126 |    7     |
| After  |  148171  |  242011  |  161025 |    7     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
196:        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..ac60192 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -193,7 +193,15 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         reservedRate = 1;

         // The minimum amount of token received if swapping
-        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));
+        uint256 _quote;
+        uint256 _slippage;
+        { // used to discard `data` variable and avoid extra stack manipulation
+            bytes calldata data = _data.metadata;
+            assembly {
+                _quote := calldataload(add(data.offset, 0x40))
+                _slippage := calldataload(add(data.offset, 0x60))
+            }
+        }
         uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

         // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L221

*Gas Savings for `JBXBuybackDelegate.uniswapV3SwapCallback`, obtained via protocol's tests: Avg 75 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28794   |  30714   |  20125  |    6     |
| After  |  28725   |  30645   |  20050  |    6     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
221:        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..b87a72a 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -218,7 +218,10 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

         // Unpack the data
-        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
+        uint256 _minimumAmountReceived;
+        assembly {
+            _minimumAmountReceived := calldataload(data.offset)
+        }

         // Assign 0 and 1 accordingly
         uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
```

## Use assembly to emit events
We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.

**Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer**.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L325

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L352

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 38 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148266  |  242106  |  161126 |    7     |
| After  |  148228  |  242068  |  161085 |    7     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
325:        emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);

352:        emit JBXBuybackDelegate_Mint(_data.projectId);
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..0836a78 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -322,7 +322,19 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             }
         }

-        emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
+        assembly {
+            let memptr := mload(0x40)
+            mstore(0x00, calldataload(0x44))
+            mstore(0x20, calldataload(0xa4))
+            mstore(0x40, _amountReceived)
+            log1(
+                0x00,
+                0x60,
+                // keccak256("JBXBuybackDelegate_Swap(uint256,uint256,uint256)")
+                0x01a4fda29d012874ff22866f5058fa420a4f8598b6f5207b9851c577c11507f7
+            )
+            mstore(0x40, memptr)
+        }
     }

     /**
@@ -349,7 +361,15 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
             _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
         );

-        emit JBXBuybackDelegate_Mint(_data.projectId);
+        assembly {
+            mstore(0x00, calldataload(0x44))
+            log1(
+                0x00,
+                0x20,
+                // keccak256("JBXBuybackDelegate_Mint(uint256)")
+                0x9dd9e06f6137ca3ab76ef60c229d956aa1d09df00d9f55800cec4e1d9cf21381
+            )
+        }
     }
```

## Use assembly to validate `msg.sender`
We can use assembly to efficiently validate msg.sender for the `didPay` and `uniswapV3SwapCallback` functions with the least amount of opcodes necessary. Additionally, we can use `xor()` instead of `iszero(eq())`, saving 3 gas. We can also potentially save gas on the unhappy path by using `scratch space` to store the error selector, potentially avoiding memory expansion costs. 

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L185

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 21 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148266  |  242106  |  161126 |    7     |
| After  |  148245  |  242085  |  161107 |    7     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
185:        if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..fc5566e 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -182,7 +182,13 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     function didPay(JBDidPayData calldata _data) external payable override {
         // Access control as minting is authorized to this delegate
-        if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
+        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal = jbxTerminal;
+        assembly {
+            if xor(caller(), _jbxTerminal) {
+                // bytes4(keccak256("JuiceBuyback_Unauthorized()"))
+                mstore(0x00, 0x84f56813)
+                revert(0x1c, 0x04)
+            }
+        }

         // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
         uint256 _tokenCount = mintedAmount;
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L218

*Gas Savings for `JBXBuybackDelegate.uniswapV3SwapCallback`, obtained via protocol's tests: Avg 12 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  28794   |  30714   |  20125  |    6     |
| After  |  28784   |  30704   |  20113  |    6     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
218:        if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..fc5566e 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -215,7 +221,13 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      */
     function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
         // Check if this is really a callback
-        if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
+        IUniswapV3Pool _pool = pool;
+        assembly {
+            if xor(caller(), _pool) {
+                // bytes4(keccak256("JuiceBuyback_Unauthorized()"))
+                mstore(0x00, 0x84f56813)
+                revert(0x1c, 0x04)
+            }
+        }

         // Unpack the data
         (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));
```

## Use assembly to efficiently read from and write to packed storage slots
The EVM reads from and writes to storage 1 word (32 bytes) at a time. Therefore, if we pack multiple values together into one storage slot, the EVM will do extra operations (bit masking/shifting) to fit each value into one word. With assembly we can use the least amount of opcodes necessary to read from and write to packed storage slots. 

**Note: This optimzation is dependent on [G-01](#state-variables-can-be-packed-to-use-fewer-storage-slots) and thus benchmarking is done using the optimization in [G-01](#state-variables-can-be-packed-to-use-fewer-storage-slots) as a baseline**.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L158-L159

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L188-L193

*Gas Savings for `JBXBuybackDelegate.payParams`, obtained via protocol's tests: Avg 18 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  6771    |  7523    |  5528   |    10    |
| After  |  6747    |  7497    |  5510   |    10    |

*Gas Savings for `JBXBuybackDelegate.didPay`, obtained via protocol's tests: Avg 33 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  148151  |  241991  |  161803 |    7     |
| After  |  148120  |  241960  |  161770 |    7     |

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
158:            mintedAmount = _tokenCount;
159:            reservedRate = _data.reservedRate;

188:            uint256 _tokenCount = mintedAmount;
189:            mintedAmount = 1;
190:
191:            // Retrieve the fc reserved rate and reset the mutex
192:            uint256 _reservedRate = reservedRate;
193:            reservedRate = 1;
```
```diff
diff --git a/contracts/JBXBuybackDelegate.sol b/contracts/JBXBuybackDelegate.sol
index 0ee751b..6455fe4 100644
--- a/contracts/JBXBuybackDelegate.sol
+++ b/contracts/JBXBuybackDelegate.sol
@@ -103,14 +103,14 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
      *
      * @dev    This is a mutex 1-x-1
      */
-    uint256 private mintedAmount = 1;
+    uint128 private mintedAmount = 1;

     /**
      * @notice The current reserved rate
      *
      * @dev    This is a mutex 1-x-1
      */
-    uint256 private reservedRate = 1;
+    uint128 private reservedRate = 1;

     /**
      * @dev No other logic besides initializing the immutables
@@ -155,8 +155,9 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
         if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
             // Pass the quote and reserve rate via a mutex
-            mintedAmount = _tokenCount;
-            reservedRate = _data.reservedRate;
+            assembly {
+                sstore(mintedAmount.slot, or(shr(128, shl(128, _tokenCount)), shl(128, calldataload(0x164))))
+            }

             // Return this delegate as the one to use, and do not mint from the terminal
             delegateAllocations = new JBPayDelegateAllocation[](1);
@@ -185,12 +186,14 @@ contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUnisw
         if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

         // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
-        uint256 _tokenCount = mintedAmount;
-        mintedAmount = 1;
-
-        // Retrieve the fc reserved rate and reset the mutex
-        uint256 _reservedRate = reservedRate;
-        reservedRate = 1;
+        uint256 _tokenCount;
+        uint256 _reservedRate;
+        assembly {
+            let storage_var := sload(mintedAmount.slot)
+            _tokenCount := shr(128, shl(128, storage_var))
+            _reservedRate := shr(128, storage_var)
+            sstore(mintedAmount.slot, or(1, shl(128, 1)))
+        }
```

## GasReport output with all optimizations applied
```js
| contracts/JBXBuybackDelegate.sol:JBXBuybackDelegate contract |                 |        |        |        |         |
|--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                              | Deployment Size |        |        |        |         |
| 1138329                                                      | 6217            |        |        |        |         |
| Function Name                                                | min             | avg    | median | max    | # calls |
| didPay                                                       | 50836           | 160778 | 146897 | 240737 | 7       |
| payParams                                                    | 1956            | 5393   | 6640   | 7378   | 10      |
| uniswapV3SwapCallback                                        | 766             | 19772  | 28316  | 30236  | 6       |
```