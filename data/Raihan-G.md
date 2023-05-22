# Gas Optimization

# Summary

|      | Issue | Instance |
|------|-------|----------|
|[G-01]|Make 3 event parameters indexed when possible|2|
|[G-02]|Amounts should be checked for 0 before calling a transfer|1|
|[G-03]|abi.encode() is less efficient than abi.encodePacked()|1|
|[G-04]|State variables can be packed into fewer storage slots|1|
|[G-05]|Use hardcode address instead address(this)|4|
|[G-06]|State variables should be cached in stack variables rather than re-reading them from storage|5|
|[G-07]|Gas savings can be achieved by changing the model for assigning value to the structure |5|
|[G-08]|Not using the named return variables when a function returns, wastes deployment gas|1|

## [G-01] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

```solidity
53    event JBXBuybackDelegate_Swap(uint256 projectId, uint256 amountEth, uint256 amountOut);
54    event JBXBuybackDelegate_Mint(uint256 projectId);
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L53-L54

## [G-2] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
232    weth.transfer(address(pool), _amountToSend);
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L232

### Recommended Mitigation Steps
```solidity
function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
    // Check if this is really a callback
    if (msg.sender != address(pool)) {
        revert JuiceBuyback_Unauthorized();
    }

    // Unpack the data
    (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

    // Assign 0 and 1 accordingly
    uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
    uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

    // Revert if slippage is too high
    if (_amountReceived < _minimumAmountReceived) {
        revert JuiceBuyback_MaximumSlippage();
    }
    
    // Check if amountToSend is greater than 0
+    require(_amountToSend > 0, "Amount to send must be greater than zero");

    // Wrap and transfer the weth to the pool
    weth.deposit{value: _amountToSend}();
    // Check if the transfer was successful
    require(weth.transfer(address(pool), _amountToSend), "Transfer failed");
}
```

## [G-03] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
268    data: abi.encode(_minimumReceivedFromSwap)
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L268

### Recommended Mitigation Steps
```solidity
try pool.swap({
        recipient: address(this),
        zeroForOne: !_projectTokenIsZero,
        amountSpecified: int256(_data.amount.value),
        sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
-        data: abi.encode(_minimumReceivedFromSwap)
+        data: abi.encodePacked(_minimumReceivedFromSwap)
    })
```
## [G-04] State variables can be packed into fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).
```solidity
  bool private immutable _projectTokenIsZero;

    /**
     * @notice The unit of the max slippage (expressed in 1/10000th)
     */
    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

    //*********************************************************************//
    // --------------------- public constant properties ------------------ //
    //*********************************************************************//

    /**
     * @notice The project token address
     * 
     * @dev In this context, this is the tokenOut
     */
    IERC20 public immutable projectToken;

    /**
     * @notice The uniswap pool corresponding to the project token-other token market
     *         (this should be carefully chosen liquidity wise)
     */
    IUniswapV3Pool public immutable pool;

    /**
     * @notice The project terminal using this extension
     */
    IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;

    /**
     * @notice The WETH contract
     */
    IWETH9 public immutable weth;

    //*********************************************************************//
    // --------------------- private stored properties ------------------- //
    //*********************************************************************//

    /**
     * @notice The amount of token created if minted is prefered
     * 
     * @dev    This is a mutex 1-x-1
     */
    uint256 private mintedAmount = 1;

    /**
     * @notice The current reserved rate
     * 
     * @dev    This is a mutex 1-x-1
     */
    uint256 private reservedRate = 1;
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63-L113
### Recommended Mitigation Steps
Note : if it's possible to reduce the type of uint than EVM use less Slots like this code:
```solidity
    bool private immutable _projectTokenIsZero;

    /**
     * @notice The unit of the max slippage (expressed in 1/10000th)
     */
    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

    //*********************************************************************//
    // --------------------- public constant properties ------------------ //
    //*********************************************************************//

    /**
     * @notice The project token address
     * 
     * @dev In this context, this is the tokenOut
     */
    IERC20 public immutable projectToken;

    /**
     * @notice The uniswap pool corresponding to the project token-other token market
     *         (this should be carefully chosen liquidity wise)
     */
    IUniswapV3Pool public immutable pool;

    /**
     * @notice The project terminal using this extension
     */
    IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;

    /**
     * @notice The WETH contract
     */
    IWETH9 public immutable weth;

    //*********************************************************************//
    // --------------------- private stored properties ------------------- //
    //*********************************************************************//

    /**
     * @notice The amount of token created if minted is prefered
     * 
     * @dev    This is a mutex 1-x-1
     */
-   uint256 private mintedAmount = 1;
+   uint128 private mintedAmount = 1;


    /**
     * @notice The current reserved rate
     * 
     * @dev    This is a mutex 1-x-1
     */
-   uint256 private reservedRate = 1;
+   uint128 private reservedRate = 1;
```

## [G-05] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
264   recipient: address(this),

294   _holder: address(this),

305   _beneficiary: address(this),

315   _holder: address(this),
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L264
### Recommended Mitigation Steps
```solidity
    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
         address thisAddress = /*Here add address*/;
        // Pass the token and min amount to receive as extra data
        try pool.swap({
-           recipient: address(this),
+           recipient: thisaddress,
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(_data.amount.value),
            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
            data: abi.encode(_minimumReceivedFromSwap)
        }) returns (int256 amount0, int256 amount1) {
            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
            _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
        } catch {
            // implies _amountReceived = 0 -> will later mint when back in didPay
            return _amountReceived;
        }

        // The amount to send to the beneficiary
        uint256 _nonReservedToken = PRBMath.mulDiv(
            _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
        );

        // The amount to add to the reserved token
        uint256 _reservedToken = _amountReceived - _nonReservedToken;

        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

        // If there are reserved token, add them to the reserve
        if (_reservedToken != 0) {
            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

            // 1 Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
            controller.burnTokensOf({
-               _holder: address(this),
+               _holder: thisaddress,
                _projectId: _data.projectId,
                _tokenCount: _reservedToken,
                _memo: "",
                _preferClaimedTokens: true
            });

            // 2 Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
            controller.mintTokensOf({
                _projectId: _data.projectId,
                _tokenCount: _amountReceived,
-               _beneficiary: address(this),
+               _beneficiary: thisaddress,
                _memo: _data.memo,
                _preferClaimedTokens: false,
                _useReservedRate: true
            });

            // 3 Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

            if (_nonReservedTokenInContract != 0) {
                controller.burnTokensOf({
-                   _holder: address(this),
+                   _holder: thisaddrss,
                    _projectId: _data.projectId,
                    _tokenCount: _nonReservedTokenInContract,
                    _memo: "",
                    _preferClaimedTokens: false
                });
            }
        }

        emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
    }
```
## [G‑06] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
224  uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
225  uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

265  zeroForOne: !_projectTokenIsZero,
267  sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
271  _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224-L225



### Recommended Mitigation Steps
```solidity
function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
    // Cache the value of _projectTokenIsZero in a local variable
+   bool _projectTokenIsZeroCached = _projectTokenIsZero;

    // Check if this is really a callback
    if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

    // Unpack the data
    (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

    // Assign 0 and 1 accordingly
-   uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
+   uint256 _amountReceived = uint256(-(_projectTokenIsZeroCached ? amount0Delta : amount1Delta));
-   uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);
+   uint256 _amountToSend = uint256(_projectTokenIsZeroCached ? amount1Delta : amount0Delta);

    // Revert if slippage is too high
    if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

    // Wrap and transfer the weth to the pool
    weth.deposit{value: _amountToSend}();
    weth.transfer(address(pool), _amountToSend);
}
```
In this updated code, we have cached the value of _projectTokenIsZero ina local variable called _projectTokenIsZeroCached. We then use this local variable to determine whether to assign amount0Delta or amount1Delta to _amountToSend and _amountReceived.

By caching the value of _projectTokenIsZero in a local variable, we avoid re-reading the state variable from storage, which can save gas costs. The cost of reading a state variable from storage is 100 gas, which can add up if the variable is accessed multiple times within a function. By caching the value in a local variable, we only need to read the value from storage once and can then use the local variable for subsequent access.

This is a simple optimization, but it can help to reduce gas costs and improve the efficiency of your contract.

### Recommended Mitigation Steps
```solidity
function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
        // Pass the token and min amount to receive as extra data
        try pool.swap({
            recipient: address(this),
-           zeroForOne: !_projectTokenIsZero,
+           zeroForOne: !_projectTokenIsZeroCached,
            amountSpecified: int256(_data.amount.value),
-           sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
+           sqrtPriceLimitX96: _projectTokenIsZeroCached ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
            data: abi.encode(_minimumReceivedFromSwap)
        }) returns (int256 amount0, int256 amount1) {
            // Swap succeded, take note of the amount of projectToken received (negative as it is an exact input)
-           _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
+           _amountReceived = uint256(-(_projectTokenIsZeroCached ? amount0 : amount1));
        } catch {
            // implies _amountReceived = 0 -> will later mint when back in didPay
            return _amountReceived;
        }

        // The amount to send to the beneficiary
        uint256 _nonReservedToken = PRBMath.mulDiv(
            _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
        );

        // The amount to add to the reserved token
        uint256 _reservedToken = _amountReceived - _nonReservedToken;

        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

        // If there are reserved token, add them to the reserve
        if (_reservedToken != 0) {
            IJBController controller = IJBController(jbxTerminal.directory().controllerOf(_data.projectId));

            // 1) Burn all the reserved token, which are in this address -> result: 0 here, 0 in reserve
            controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
                _tokenCount: _reservedToken,
                _memo: "",
                _preferClaimedTokens: true
            });

            // 2) Mint the reserved token with this address as beneficiary -> result: _amountReceived-reserved here, reservedToken in reserve
            controller.mintTokensOf({
                _projectId: _data.projectId,
                _tokenCount: _amountReceived,
                _beneficiary: address(this),
                _memo: _data.memo,
                _preferClaimedTokens: false,
                _useReservedRate: true
            });

            // 3) Burn the non-reserve token which are now left in this address (can be 0) -> result: 0 here, reservedToken in reserve
            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

            if (_nonReservedTokenInContract != 0) {
                controller.burnTokensOf({
                    _holder: address(this),
                    _projectId: _data.projectId,
                    _tokenCount: _nonReservedTokenInContract,
                    _memo: "",
                    _preferClaimedTokens: false
                });
            }
        }

        emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);
    }
```


## [G-07] Gas savings can be achieved by changing the model for assigning value to the structure 
Here's an example of how you can use the memory keyword with a struct:

```
struct MyStruct {
    uint256 variable1;
    uint256 variable2;
}

function myFunction() public {
    MyStruct storge/memory myStruct;
    myStruct.variable1 = 123;
    myStruct.variable2 = 456;

    // do something with myStruct in memory
}
```
```solidity
263    ry pool.swap({
            recipient: address(this),
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(_data.amount.value),
            sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
            data: abi.encode(_minimumReceivedFromSwap)
        }) returns (int256 amount0, int256 amount1) {

293      controller.burnTokensOf({
                _holder: address(this),
                _projectId: _data.projectId,
                _tokenCount: _reservedToken,
                _memo: "",
                _preferClaimedTokens: true
            });

302     controller.mintTokensOf({
                _projectId: _data.projectId,
                _tokenCount: _amountReceived,
                _beneficiary: address(this),
                _memo: _data.memo,
                _preferClaimedTokens: false,
                _useReservedRate: true
            });

314     if (_nonReservedTokenInContract != 0) {
                controller.burnTokensOf({
                    _holder: address(this),
                    _projectId: _data.projectId,
                    _tokenCount: _nonReservedTokenInContract,
                    _memo: "",
                    _preferClaimedTokens: false
                });
            }

338     controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amount,
            _beneficiary: _data.beneficiary,
            _memo: _data.memo,
            _preferClaimedTokens: _data.preferClaimedTokens,
            _useReservedRate: true
        });            

```

## [G-08]Not using the named return variables when a function returns, wastes deployment gas
When a function returns multiple values without named return variables, it creates a temporary variable to hold the returned values, which can increase the deployment gas cost
```solidity
359    function supportsInterface(bytes4 _interfaceId) external pure override returns (bool) {
       return _interfaceId == type(IJBFundingCycleDataSource).interfaceId
            || _interfaceId == type(IJBPayDelegate).interfaceId;
    }
```
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L359-L362
### Recommended Mitigation Steps
```solidity
+    function supportsInterface(bytes4 _interfaceId) external pure override returns (bool x) {
+       x = _interfaceId == type(IJBFundingCycleDataSource).interfaceId
            || _interfaceId == type(IJBPayDelegate).interfaceId;
    }
```
In this version, we use a named return variable x, which is assigned the boolean value and returned directly without the need for a temporary variable. This can help to optimize gas usage by avoiding the overhead of creating and managing a temporary variable.