### QA Report Issues List

- [x] **Low 01** → Missing `deadline` checks allow pending transactions to be maliciously executed
- [x] **Low 02** → Transaction will be reverted even if there is a `_minimumAmountReceived ` amount
- [x] **Low 03** → Use oracle price for slippage control instead manuel slippage control
- [x] **Low 04**→ Uniswap's SwapRouter doesn't refund unspent ETH in partial swaps
- [x] **Low 05**→ The size of the `_data.metadata` struct is not checked, which may cause errors
- [x] **Low 06** → Update code to not support swaps entirely within 0 liquidity region
- [x] **Low 07** → First inheritance should be Ownable.sol
- [x] **Low 08** → In the real world, using underutilized dependencies is risky
- [x] **Low 09**→ `pool` address variable should be changeable architecture
- [x] **Low 10**→ In the `pool` address variable, precautions should be taken against incorrect address definition
- [x] **Low 11**→ Missing Event for  initialize
- [x] **Low 12**→ Project Upgrade and Stop Scenario should be
- [x] **Non-Critical  13**→ Tokens accidentally sent to the contract cannot be recovered
- [x] **Non-Critical  14**→ `_projectTokenIsZero` variable name is confused
- [x] **Non-Critical  15**→ Use underscores for number literals
- [x] **Non-Critical  16**→`Ownable`  is indeed an unused import 
- [x] **Non-Critical  17**→ `payParams` function architecture in terms of liquidity management can be change


### [Low-01] Missing deadline checks allow pending transactions to be maliciously executed


it should provide their users with an option to limit the execution of their pending actions, such as swaps or adding and removing liquidity. The most common solution is to include a deadline timestamp as a parameter

If such an option is not present, users can unknowingly perform bad trades.An even worse way this issue can be maliciously exploited is through MEV.

```solidity
contracts/JBXBuybackDelegate.sol:
  349     */
  350:   function _swap(
  351:     JBDidPayData calldata _data,
  352:     uint256 _minimumReceivedFromSwap,
  353:     uint256 _reservedRate
  354:   ) internal returns (uint256 _amountReceived) {
  355:     // Pass the token and min amount to receive as extra data
  356:     try
  357:       pool.swap({
  358:         recipient: address(this),
  359:         zeroForOne: !_projectTokenIsZero,
  360:         amountSpecified: int256(_data.amount.value),
  361:         sqrtPriceLimitX96: _projectTokenIsZero
  362:           ? TickMath.MAX_SQRT_RATIO - 1
  363:           : TickMath.MIN_SQRT_RATIO + 1,
  364:         data: abi.encode(_minimumReceivedFromSwap)
  365:       })
  366:     returns (int256 amount0, int256 amount1) {
  367:       // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
  368:       _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
  369:     } catch {
  370:       // implies _amountReceived = 0 -> will later mint when back in didPay
  371:       return _amountReceived;
  372:     }
  373: 
  374:     // The amount to send to the beneficiary
  375:     uint256 _nonReservedToken = PRBMath.mulDiv(
  376:       _amountReceived,
  377:       JBConstants.MAX_RESERVED_RATE - _reservedRate,
  378:       JBConstants.MAX_RESERVED_RATE
  379:     );
  380: 
  381:     // The amount to add to the reserved token
  382:     uint256 _reservedToken = _amountReceived - _nonReservedToken;
  383: 
  384:     // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
  385:     if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);
  386: 
  387:     // If there are reserved token, add them to the reserve
  388:     if (_reservedToken != 0) {
  389:       IJBController controller = IJBController(
  390:         jbxTerminal.directory().controllerOf(_data.projectId)
  391:       );
  392: 
  393:       // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
  394:       controller.burnTokensOf({
  395:         _holder: address(this),
  396:         _projectId: _data.projectId,
  397:         _tokenCount: _reservedToken,
  398:         _memo: '',
  399:         _preferClaimedTokens: true
  400:       });
  401: 
  402:       // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
  403:       controller.mintTokensOf({
  404:         _projectId: _data.projectId,
  405:         _tokenCount: _amountReceived,
  406:         _beneficiary: address(this),
  407:         _memo: _data.memo,
  408:         _preferClaimedTokens: false,
  409:         _useReservedRate: true
  410:       });
  411: 
  412:       // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
  413:       uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;
  414: 
  415:       if (_nonReservedTokenInContract != 0) {
  416:         controller.burnTokensOf({
  417:           _holder: address(this),
  418:           _projectId: _data.projectId,
  419:           _tokenCount: _nonReservedTokenInContract,
  420:           _memo: '',
  421:           _preferClaimedTokens: false
  422:         });
  423:       }
  424:     }
  425: 
  426:     emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
  427:   }

```

## Recommendation

Introduce a deadline parameter to all functions which potentially perform a swap on the user's behalf.

### [Low-02] Transaction will be reverted even if there is a `_minimumAmountReceived ` amount

In uniswapV3SwapCallback() function checks `_minimumAmountReceived` value for slippage control but operation reverts even if specified value


```diff
contracts/JBXBuybackDelegate.sol:
  286     */
  287:   function uniswapV3SwapCallback(
  288:     int256 amount0Delta,
  289:     int256 amount1Delta,
  290:     bytes calldata data
  291:   ) external override {
  292:     // Check if this is really a callback
  293:     if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
  294: 
  295:     // Unpack the data
  296:     uint256 _minimumAmountReceived = abi.decode(data, (uint256));
  297: 
  298:     // Assign 0 and 1 accordingly
  299:     uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
  300:     uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
  301: 
  302:     // Revert if slippage is too high
- 303:     if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
+          if (_amountReceived <= _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
  304: 
  305:     // Wrap and transfer the weth to the pool
  306:     weth.deposit{value: _amountToSend}();
  307:     weth.transfer(address(pool), _amountToSend);
  308:   }

```



### [Low-03] Use oracle price for slippage control instead manuel slippage control

Slippage is checked by uniswapV3SwapCallback() function in line 298-303 , the slippage control done here is done manually and not based on the price obtained through an oracle, use an oracle to define appropriate slippage control

Using an Oracle for Slippage Control: This approach involves integrating with an external oracle service that provides real-time price data. The oracle can help you determine the appropriate slippage control by fetching the current market price of the assets involved in the swap. You can then use this information to calculate the acceptable slippage percentage
Pros:
-Provides dynamic and accurate slippage control based on real-time market data.
-Allows for automatic adjustment of slippage control as market conditions change.
-Reduces the risk of slippage due to sudden price movements.

You could calculate the `SQRT_RATIO` according to oracle prices


```solidity

contracts/JBXBuybackDelegate.sol:
  359      // Pass the token and min amount to receive as extra data
  360:     try
  361:       pool.swap({
  362:         recipient: address(this),
  363:         zeroForOne: !_projectTokenIsZero,
  364:         amountSpecified: int256(_data.amount.value),
  365:         sqrtPriceLimitX96: _projectTokenIsZero
  366:           ? TickMath.MAX_SQRT_RATIO - 1   // Token 0 -> Token1
  367:           : TickMath.MIN_SQRT_RATIO + 1,  // Token 1 -> Token0
  368:         data: abi.encode(_minimumReceivedFromSwap)
  369:       })


contracts/JBXBuybackDelegate.sol:
  301  
  302:     // Revert if slippage is too high
  303:     if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
```

## Recommendation

Use oracle for slippage control.


### [Low-04] Uniswap's SwapRouter doesn't refund unspent ETH in partial swaps

The bug is that SwapRouter doesn’t refund unspent ETH in Uniswapv3. If you sell ETH and set a limit price and it’s reached during the swap, then the input amount will be reduced, but you have already sent a bigger amount. The remaining amount will be left in the router.

This is a bug reported in detail by Jeiwan. Since the Uniswapv3 swap transaction is used in the JBXBuybackDelegate.sol contract, users are within this scope, information should be added to the Documents and NatSpec swap function to be warned about this issue.

Although the Uniswap v3 contract is out of scope, the issue is important because the swap function in the contract interacts directly, so it is classified as NC


https://jeiwan.net/posts/public-bug-report-uniswap-swaprouter/
https://github.com/Jeiwan/uniswapv3-unrefunded-eth-poc



**Recommendation:**

MultiCall allows users to do multiple function calls in one transaction, thus Uniswap suggests that users should use MultiCall to swap tokens and call `refundETH` afterwards

For this reason, if the user will lose ETH in the swap process, an architecture should be added where he can get his ETH with MultiCall.



### [Low-05] The size of the `_data.metadata` struct is not checked, which may cause errors

Does not check the length of the metadata variable before calling the `abi.decode` in function

```solidity
contracts/JBXBuybackDelegate.sol:
  190:     (, , uint256 _quote, uint256 _slippage) = abi.decode( 
  191:       _data.metadata,
  192:       (bytes32, bytes32, uint256, uint256)
  193:     );
```

This means that a malicious metadata variable that is longer than the expected length could be sent, causing `abi.decode` to crash or return incorrect values

In current solidity, it is neither possible to check validity of the format before sending to abi.decode, nor possible to try/catch on abi.decode itself, hence making the sanity check of input data almost impossible (unless writing your own format check

To fix this vulnerability, the smart contract should check the length of the metadata variable before calling abi.decode

```diff
contracts/JBXBuybackDelegate.sol:
  171     */
  172:   function payParams(
  173:     JBPayParamsData calldata _data
  174:   )
  175:     external
  176:     override
  177:     returns (
  178:       uint256 weight, 
  179:       string memory memo,
  180:       JBPayDelegateAllocation[] memory delegateAllocations
  181:     )
  182:   {
  183:     // Find the total number of tokens to mint, as a fixed point number with 18 decimals
  185:     uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
  186: 
  187:     // Unpack the quote from the pool, given by the frontend
+             if (bytes(metadata).length != 128) revert JuiceBuyback_Unauthorized();
  190:     (, , uint256 _quote, uint256 _slippage) = abi.decode( 
  191:       _data.metadata,
  192:       (bytes32, bytes32, uint256, uint256)
  193:     );
  194: 
  195:     // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
  197:     if (_tokenCount < _quote - ((_quote * _slippage) / SLIPPAGE_DENOMINATOR)) {
  198:       // Pass the quote and reserve rate via a mutex
  199:       mintedAmount = _tokenCount;
  200:       reservedRate = _data.reservedRate;
  201: 
  202:       // Return this delegate as the one to use, and do not mint from the terminal
  203:       delegateAllocations = new JBPayDelegateAllocation[](1);
  204:       delegateAllocations[0] = JBPayDelegateAllocation({
  205:         delegate: IJBPayDelegate(this),
  206:         amount: _data.amount.value
  207:       });
  208: 
  209:       return (0, _data.memo, delegateAllocations);
  210:     }
  211: 
  212:     // If minting, do not use this as delegate
  213:     return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
  214:   }
```


### [Low-06] Update code to not support swaps entirely within 0 liquidity region

Added part of the code below; This part determines which token is being swapped in and which one is being swapped out. It initializes two variables: isUnderlyingIn and amountToPay. If amount0Delta is greater than zero, it means token0 is being swapped in, so amountToPay is set to amount0Delta, and the variable isUnderlyingIn is assigned the value of token0IsUnderlying.

On the other hand, if amount0Delta is not greater than zero, it means token1 is being swapped in. In this case, it checks that amount1Delta is greater than zero (to ensure the swap doesn't happen entirely within a 0-liquidity region). If the condition is met, amountToPay is set to amount1Delta, and isUnderlyingIn is assigned the negation of token0IsUnderlying.

In summary ,it means that swaps where both token0 and token1 are being withdrawn entirely from the liquidity pool are not allowed. It's a design decision to prevent potential issues that could arise from such swaps.


```diff

contracts/JBXBuybackDelegate.sol:
  286     */
  287:   function uniswapV3SwapCallback(
  288:     int256 amount0Delta,
  289:     int256 amount1Delta,
  290:     bytes calldata data
  291:   ) external override {
  292:     // Check if this is really a callback
  293:     if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

+           bool isUnderlyingIn;
+           uint256 amountToPay;
+           if (amount0Delta > 0) {
+          	amountToPay = uint256(amount0Delta);
+          	isUnderlyingIn = token0IsUnderlying;
+            } else {
+               require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
+               amountToPay = uint256(amount1Delta);
+              isUnderlyingIn = !(token0IsUnderlying);
+           }

  294: 
  295:     // Unpack the data
  296:     uint256 _minimumAmountReceived = abi.decode(data, (uint256));
  297: 
  298:     // Assign 0 and 1 accordingly
  299:     uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
  300:     uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
  301: 
  302:     // Revert if slippage is too high
  303:     if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
  304: 
  305:     // Wrap and transfer the weth to the pool
  306:     weth.deposit{value: _amountToSend}();
  307:     weth.transfer(address(pool), _amountToSend);
  308:   }


```


### [Low-07] First inheritance should be Ownable.sol

While `JBXBuybackDelegate` contract defines inheritance, it defines the most critical `Ownable` contract for access control at the end, it is correct for security to be defined from the first row as best practice.

Note ; Ownable  is indeed an unused import (leftover from previous version iirc) , however it is used here and this is potentially a security vulnerability. Not being used in the project and the vulnerability here are completely separate issues.

```diff
contracts/JBXBuybackDelegate.sol:
  49  
  50: contract JBXBuybackDelegate is
+         Ownable,
  51:   IJBFundingCycleDataSource,
  52:   IJBPayDelegate,
  53:   IUniswapV3SwapCallback,
- 54:   Ownable
  55: {
```


### [Low-08] In the real world, using underutilized dependencies is risky


```solidity
contracts/JBXBuybackDelegate.sol:
18: import '@paulrberg/contracts/math/PRBMath.sol’;
```

https://www.npmjs.com/package/@paulrberg/contracts?activeTab=versions


I recommend the audited and battle tested OpenZeppelin library 's Math library


### [Low-09] `pool` address variable should be changeable architecture.

In the project, values such as the platform token address are fixed and do not change later, but the `UniswapV3Pool` address may change in the future for more than one reason (the liquidity in the pool is very low and the pool is shallow, the liquidity is high in other pools and therefore it is more preferred), but `pool` address is not updateable by owner

```solidity

contracts/JBXBuybackDelegate.sol:
   99  
  100:   /**
  101:    * @notice The uniswap pool corresponding to the project token-other token market
  102:    *         (this should be carefully chosen liquidity wise)
  103:    */
  104:   IUniswapV3Pool public immutable pool;

```

Recommendation Step;

Design the `pool` address in a mutable architecture ;

```diff

- IUniswapV3Pool public immutable pool;
+ IUniswapV3Pool public pool;

+ function updatePool (_update address) {
+   if(_update == address(0)) revert addressCheckZero();
+   pool = _update;
+ }

```



### [Low-10] In the `pool` address variable, precautions should be taken against incorrect address definition.

The fix below gives an advantage in 2 separate parts

1- It prevents the incorrect definition of the pool address in the constructor
2- It prevents the pool address from being defined as 0 address in the constructor

```diff
+ import {UniswapHelpers} from "./libraries/UniswapHelpers.sol";
- IUniswapV3Pool public immutable pool;
+ IUniswapV3Pool public pool;


contracts/JBXBuybackDelegate.sol:
  140     */
  141:   constructor(
  142:     IERC20 _projectToken,
  143:     IWETH9 _weth,
  144:     IUniswapV3Pool _pool,
  145:     IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
  146:   ) {
  147:     projectToken = _projectToken;
- 148:     pool = _pool;
+             _setPool(_pool);
  149:     jbxTerminal = _jbxTerminal;
  150:     _projectTokenIsZero = address(_projectToken) < address(_weth);
  151:     weth = _weth;
  152:   }


+  /// @notice Updates `pool`
+  /// @dev reverts if new pool does not have same token0 and token1 as `pool`
+   /// @dev if pool = address(0), does NOT check that tokens match papr and underlying
+  function _setPool(address _pool) internal {
+       if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();
+       if (!UniswapHelpers.isUniswapPool(_pool)) revert InvalidUniswapV3Pool();
+       pool = _pool;
+       emit SetPool(_pool);
+  }
```


### [Low-11] Missing Event for  initialize

**Context:**
```solidity
contracts/JBXBuybackDelegate.sol:
  136     */
  137:   constructor(
  138:     IERC20 _projectToken,
  139:     IWETH9 _weth,
  140:     IUniswapV3Pool _pool,
  141:     IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
  142:   ) {
  143:     projectToken = _projectToken;
  144:     pool = _pool;
  145:     jbxTerminal = _jbxTerminal;
  146:     _projectTokenIsZero = address(_projectToken) < address(_weth);
  147:     weth = _weth;
  148:   }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit


### [Low-12] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [Non Critical-13] Tokens accidentally sent to the contract cannot be recovered

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity

 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```

### [Non Critical-14] `_projectTokenIsZero` variable name is confused

```solidity
contracts/JBXBuybackDelegate.sol:
  78     */
  79:   bool private immutable _projectTokenIsZero;
```

The variable name can be misleading as `_projectTokenIsZero` does not necessarily indicate zero



A possible alternative name to `_projectTokenIsZero` would be `_projectTokenIsLessThanWETH`. This name provides more clarity by stating that the comparison checks whether `_projectToken` is numerically less than `_weth` 


```diff
contracts/JBXBuybackDelegate.sol:
-  79:   bool private immutable _projectTokenIsZero;
+ 79:   bool private immutable _projectTokenIsLessThanWETH;

```


### [Non Critical -15] Use underscores for number literals

```diff
contracts/JBXBuybackDelegate.sol:
-  84:   uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
+ 84:   uint256 private constant SLIPPAGE_DENOMINATOR = 10_000;
```
**Description:**
 There is   occasions where certain number have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

**Recommendation:**
Consider using underscores for number literals to improve its readability.


### [Non Critical-16] `Ownable`  is indeed an unused import 

Ownable  is indeed an unused import (leftover from previous version iirc)


```diff

- 15: import '@openzeppelin/contracts/access/Ownable.sol';

contracts/JBXBuybackDelegate.sol:
  49  
  50: contract JBXBuybackDelegate is
  51:   IJBFundingCycleDataSource,
  52:   IJBPayDelegate,
  53:   IUniswapV3SwapCallback,
- 54:   Ownable
  55: {
```

### [Non Critical-17] `payParams` function architecture in terms of liquidity management can be change


The variable `uint256 _tokenCount` finds the total number of tokens for mint in the `payParams` function.

However, due to architecture; If the amount swapped is bigger than the lowest received when minting, use the swap pathway

Here, all or nothing algorithm is applied, it is checked whether `_tokenCount` is fully provided, whereas in terms of effective liquidity management; partial `_tokenCount` can be used and the remaining amount can be minted, this is a detailed but effective architectural method


```solidity
contracts/JBXBuybackDelegate.sol:
  171     */
  172:   function payParams(
  173:     JBPayParamsData calldata _data
  174:   )
  175:     external
  176:     override
  177:     returns (
  178:       uint256 weight, 
  179:       string memory memo,
  180:       JBPayDelegateAllocation[] memory delegateAllocations
  181:     )
  182:   {
  185:     uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
  186: 
  190:     (, , uint256 _quote, uint256 _slippage) = abi.decode( 
  191:       _data.metadata,
  192:       (bytes32, bytes32, uint256, uint256)
  193:     );
  194: 
  200:     if (_tokenCount < _quote - ((_quote * _slippage) / SLIPPAGE_DENOMINATOR)) {
  201:       // Pass the quote and reserve rate via a mutex  
  202:       mintedAmount = _tokenCount;
  203:       reservedRate = _data.reservedRate;
  204: 
  205:       // Return this delegate as the one to use, and do not mint from the terminal
  206:       delegateAllocations = new JBPayDelegateAllocation[](1);
  207:       delegateAllocations[0] = JBPayDelegateAllocation({
  208:         delegate: IJBPayDelegate(this),
  209:         amount: _data.amount.value
  210:       });
  211: 
  212:       return (0, _data.memo, delegateAllocations);
  213:     }
  214: 
  215:     // If minting, do not use this as delegate
  216:     return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
  218:   }

```

