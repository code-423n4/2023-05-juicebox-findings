###  Gas Optimizations List
 --------------------------------------------------
- [x] **Gas 01** → Missing `zero-address` check in `constructor`
- [x] **Gas 02** → Defining WETH address directly saves gas
- [x] **Gas 03** → ``Ownable.sol`` is really an unused import
- [x] **Gas 04** → Setting the state variable to its default value using the 'delete' keyword saves gas
- [x] **Gas 05** → Using bools for storage incurs overhead
- [x] **Gas 06** → Empty blocks should be removed or emit something

###  [G-01] Missing `zero-address` check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

4 results - 1 file:

```solidity
contracts\JBXBuybackDelegate.sol:

  118:     constructor(
  119:         IERC20 _projectToken,
  120:         IWETH9 _weth,
  121:         IUniswapV3Pool _pool,
  122:         IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
  123:     ) {
  124:         projectToken = _projectToken;
  125:         pool = _pool;
  126:         jbxTerminal = _jbxTerminal;
  127:         _projectTokenIsZero = address(_projectToken) < address(_weth);
  128:         weth = _weth;
  129:     }

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129


### [G-02] Defining WETH address directly saves gas

WETH is a wrapping Ether contract with a specific address on the Ethereum network. Here, it is healthier to define the weth address directly as a ``constant`` variable.

WETH Address: 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2

Advantages of direct definition of WETH contract:

- It saves gas,
- Prevents incorrect argument definition,
- Prevents execution and re-signature issues on a different chain,

In addition, gas will be saved as there will be no need to use WETH address as constructor argument.

```diff
contracts\JBXBuybackDelegate.sol:

   92      /**
   93:      * @notice The WETH contract
   94       */
-  95:     IWETH9 public immutable weth;
+  95:     IWETH9 public constant WETH = IWETH9(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2);

  118:     constructor(
  119:         IERC20 _projectToken,
- 120:         IWETH9 _weth,
  121:         IUniswapV3Pool _pool,
  122:         IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
  123:     ) {
  124:         projectToken = _projectToken;
  125:         pool = _pool;
  126:         jbxTerminal = _jbxTerminal;
- 127:         _projectTokenIsZero = address(_projectToken) < address(_weth);
+ 127:         _projectTokenIsZero = address(_projectToken) < address(WETH);
- 128:         weth = _weth;
  129:     }

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L95
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129


###  [G-03] ``Ownable.sol`` is really an unused import

``Ownable.sol`` is an unused import (remaining from previous iirc version), removing it will save gas.


```diff
contracts\JBXBuybackDelegate.sol:
 
- 15: import "@openzeppelin/contracts/access/Ownable.sol";


- 39: contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {
+ 39: contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback {

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L15

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L39


###  [G-04] Setting the state variable to its default value using the 'delete' keyword saves gas

The value of the `reservedRate` state variable is 1 by default. For this reason, a small amount of gas can be saved by using the "delete" keyword as shown in the code below.

1 result - 1 file:

```diff
contracts\JBXBuybackDelegate.sol:

  113:     uint256 private reservedRate = 1;


  183:     function didPay(JBDidPayData calldata _data) external payable override {
  184:         // Access control as minting is authorized to this delegate
  185:         if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
  186: 
  187:         // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
  188:         uint256 _tokenCount = mintedAmount;
- 189:         mintedAmount = 1;
+ 189:         delete mintedAmount;
  190:         
  191: 
  192:         // Retrieve the fc reserved rate and reset the mutex
  193:         uint256 _reservedRate = reservedRate;
- 194:         reservedRate = 1;
+ 194:         delete reservedRate;

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L189
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L194


###  [G-05] Using bools for storage incurs overhead

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

Reference: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27


1 result - 1 file:

```solidity
contracts\JBXBuybackDelegate.sol:
  62       */
  63:     bool private immutable _projectTokenIsZero;

```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63


### [G-06] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.


1 result - 1 file:
```solidity
contracts\JBXBuybackDelegate.sol:
  234  
  235:     function redeemParams(JBRedeemParamsData calldata _data)
  236:         external
  237:         override
  238:         returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
  239:     {}

```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L239

