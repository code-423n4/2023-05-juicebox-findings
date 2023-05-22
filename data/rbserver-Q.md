## QA REPORT

| |Issue|
|-|:-|
| [01] | `projectToken.transfer(_data.projectId, _data.beneficiary, _nonReservedToken)` CAN BE EXECUTED INSTEAD OF `projectToken.transfer(_data.beneficiary, _nonReservedToken)` IN `JBXBuybackDelegate._swap` FUNCTION |
| [02] | `JBXBuybackDelegate` CONTRACT'S CONSTRUCTOR CAN BE UPDATED TO CHECK IF `_pool` IS FOR `_projectToken` |
| [03] | THERE IS NO SWAP DEADLINE FOR CALLING `JBXBuybackDelegate._swap` FUNCTION |
| [04] | `pool` IS NOT UPDATABLE IN `JBXBuybackDelegate` CONTRACT |
| [05] | MISSING `address(0)` CHECKS FOR CRITICAL CONSTRUCTOR INPUTS |
| [06] | `_nonReservedToken` CAN BE USED, INSTEAD OF `_nonReservedTokenInContract`, AS `_tokenCount` FOR CALLING `controller.burnTokensOf` IN `JBXBuybackDelegate._swap` FUNCTION |
| [07] | `IJBFundingCycleBallot` INTERFACE IS NOT USED IN `JBXBuybackDelegate` CONTRACT |
| [08] | `SLIPPAGE_DENOMINATOR` CAN BE ADDED TO AND IMPORTED FROM `JBConstants` LIBRARY |
| [09] | UNDERSCORE CAN BE ADDED FOR `SLIPPAGE_DENOMINATOR` |
| [10] | REDUNDANT NAMED RETURN FOR `delegateAllocations` |
| [11] | `control` IS MISTYPED AS `controle` |
| [12] | SOLIDITY VERSION `0.8.20` CAN BE USED |

## [01] `projectToken.transfer(_data.projectId, _data.beneficiary, _nonReservedToken)` CAN BE EXECUTED INSTEAD OF `projectToken.transfer(_data.beneficiary, _nonReservedToken)` IN `JBXBuybackDelegate._swap` FUNCTION
When calling the following `JBXBuybackDelegate._swap` function, `if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken)` is executed. In the `JBXBuybackDelegate` contract, if `projectToken` is set to a `JBToken` contract, which has `projectId` that is not `_data.projectId` used for calling the `JBXBuybackDelegate._swap` function, and `pool` is set to a pool for swapping such `projectToken` due to an intentional or unintentional misconfiguration, then calling the `JBXBuybackDelegate._swap` function can transfer `_nonReservedToken` amount of such `projectToken` to `_data.beneficiary`. No matter if such misconfiguration is intentional and malicious or unintentional and accidental, the beneficiary specified by the user can receive `projectToken` that is not related to the corresponding project and can be worthless after such user pays the project. To safeguard against such misconfiguration, please consider executing `projectToken.transfer(_data.projectId, _data.beneficiary, _nonReservedToken)` instead of `projectToken.transfer(_data.beneficiary, _nonReservedToken)` in the `JBXBuybackDelegate._swap` function because the `JBToken.transfer(uint256 _projectId, address _to, uint256 _amount)` function would execute `if (projectId != 0 && _projectId != projectId) revert BAD_PROJECT()`.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326
```solidity
    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
        ...

        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

        ...
    }
```

```solidity
File: node_modules\@jbx-protocol\juice-contracts-v3\contracts\JBToken.sol
182:   function transfer(
183:     uint256 _projectId,
184:     address _to,
185:     uint256 _amount
186:   ) external override {
187:     // Can't transfer for a wrong project.
188:     if (projectId != 0 && _projectId != projectId) revert BAD_PROJECT();
189: 
190:     transfer(_to, _amount);
191:   }
```

## [02] `JBXBuybackDelegate` CONTRACT'S CONSTRUCTOR CAN BE UPDATED TO CHECK IF `_pool` IS FOR `_projectToken`
Calling the following constructor in the `JBXBuybackDelegate` contract would set `projectToken` and `pool`. Yet, if not being careful enough, it is possible that `pool` can be set to a pool that is not for `projectToken`. If this occurs, calling the `JBXBuybackDelegate._swap` function can transfer amount of a token that is not `projectToken` to the `JBXBuybackDelegate` contract but revert when executing `projectToken.transfer(_data.beneficiary, _nonReservedToken)` since the `JBXBuybackDelegate` contract does not own any `projectToken`. To prevent this unexpected behavior from happening, please consider checking if `_pool` is for `_projectToken` in the `JBXBuybackDelegate` contract's constructor and reverting such constructor if `_pool` is not for `_projectToken`.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129
```solidity
    constructor(
        IERC20 _projectToken,
        IWETH9 _weth,
        IUniswapV3Pool _pool,
        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
    ) {
        projectToken = _projectToken;
        pool = _pool;
        ...
    }
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326
```solidity
    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
        // Pass the token and min amount to receive as extra data
        try pool.swap({
            recipient: address(this),
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

        ...

        // Send the non-reserved token to the beneficiary (if any / reserved rate is not max)
        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

        ...
    }
```

## [03] THERE IS NO SWAP DEADLINE FOR CALLING `JBXBuybackDelegate._swap` FUNCTION
Calling the following `JBXBuybackDelegate._swap` function makes the `pool.swap` function call, which does not enforce a deadline for the swap. It is possible that the transaction for the swap remains pending in the mempool for a long time, such as due to the accompanying transaction fee being too low. Later, when a miner finally includes such transaction, the amount of the project token being swapped out can be much less than that if such transaction could be executed when it was just sent, such as because of the MEV bots' operations for attempting to manipulate the pool's price. Although the `JBXBuybackDelegate.uniswapV3SwapCallback` function does execute `if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage()`, it can be less optimal for the user without enforcing the swap deadline since the amount of the project token being swapped out at the time when the transaction is finally executed can be equal to or just higher than `_minimumAmountReceived` while such token amount being swapped out could be much higher than `_minimumAmountReceived` if such transaction could be executed when it was just sent. Hence, to optimize the swap functionality and prevent the MEV bots and other malicious actors from taking advantages of the transaction that calls the `JBXBuybackDelegate._swap` function, `_data.metadata` for the `JBXBuybackDelegate.didPay` function can be updated to allow the user to encode a specified deadline for the swap, and the `JBXBuybackDelegate.uniswapV3SwapCallback` function can be updated to revert if `block.timestamp` has passed such swap deadline.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326
```solidity
    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
        // Pass the token and min amount to receive as extra data
        try pool.swap({
            recipient: address(this),
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

        ...
    }
```

## [04] `pool` IS NOT UPDATABLE IN `JBXBuybackDelegate` CONTRACT
As shown below, `pool` is an immutable, which is assigned after calling the `JBXBuybackDelegate` contract's constructor. It is possible that the liquidity of the assigned pool becomes much smaller in the future. However, when this happens, `pool` cannot be updated to a relevant pool that has a higher liquidity. To avoid continuing using a pool that has a small liquidity, please consider changing `pool` to not be an immutable and adding a function, which should be only callable by the corresponding project's admin, for updating `pool`.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L85
```solidity
    IUniswapV3Pool public immutable pool;
```

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129
```solidity
    constructor(
        IERC20 _projectToken,
        IWETH9 _weth,
        IUniswapV3Pool _pool,
        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
    ) {
        ...
        pool = _pool;
        ...
    }
```

## [05] MISSING `address(0)` CHECKS FOR CRITICAL CONSTRUCTOR INPUTS
To prevent unintended behaviors, critical constructor inputs should be checked against `address(0)`. `address(0)` checks are missing for `_projectToken`, `_weth`, `_pool`, and `_jbxTerminal` in the following constructor. Please consider checking them.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129
```solidity
    constructor(
        IERC20 _projectToken,
        IWETH9 _weth,
        IUniswapV3Pool _pool,
        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
    ) {
        projectToken = _projectToken;
        pool = _pool;
        jbxTerminal = _jbxTerminal;
        _projectTokenIsZero = address(_projectToken) < address(_weth);
        weth = _weth;
    }
```

## [06] `_nonReservedToken` CAN BE USED, INSTEAD OF `_nonReservedTokenInContract`, AS `_tokenCount` FOR CALLING `controller.burnTokensOf` IN `JBXBuybackDelegate._swap` FUNCTION
In the following `JBXBuybackDelegate._swap` function, `uint256 _reservedToken = _amountReceived - _nonReservedToken` is executed before executing `uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken`. Since `_amountReceived`, `_nonReservedToken`, and `_reservedToken` are not changed between these two operations, `_nonReservedTokenInContract` is basically same as `_nonReservedToken`. Hence, to make the code more efficient, please consider using `_nonReservedToken`, instead of calculating and using `_nonReservedTokenInContract`, as `_tokenCount` for calling `controller.burnTokensOf`.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L258-L326
```solidity
    function _swap(JBDidPayData calldata _data, uint256 _minimumReceivedFromSwap, uint256 _reservedRate)
        internal
        returns (uint256 _amountReceived)
    {
        ...

        // The amount to add to the reserved token
        uint256 _reservedToken = _amountReceived - _nonReservedToken;

        ...

        // If there are reserved token, add them to the reserve
        if (_reservedToken != 0) {
            ...
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

        ...
    }
```

## [07] `IJBFundingCycleBallot` INTERFACE IS NOT USED IN `JBXBuybackDelegate` CONTRACT
The `IJBFundingCycleBallot` interface is not used in the `JBXBuybackDelegate` contract, please consider removing the import statement for it for better code efficiency.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L8
```solidity
import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleBallot.sol";
```

## [08] `SLIPPAGE_DENOMINATOR` CAN BE ADDED TO AND IMPORTED FROM `JBConstants` LIBRARY
For a better code organization, the following `SLIPPAGE_DENOMINATOR` constant can be added to and imported from the `JBConstants` library.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68
```solidity
    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
```

## [09] UNDERSCORE CAN BE ADDED FOR `SLIPPAGE_DENOMINATOR`
It is a common practice to separate each 3 digits in a number by an underscore to improve code readability. The following `SLIPPAGE_DENOMINATOR` constant does not use any underscores; please consider adding an underscore for it.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68
```solidity
    uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
```

## [10] REDUNDANT NAMED RETURN FOR `delegateAllocations`
( Please note that the following instance is not found in https://gist.github.com/itsmetechjay/2efc963de59bcad62e69de48171d10ca#n05-adding-a-return-statement-when-the-function-defines-a-named-return-variable-is-redundant. )

When a function has unused named returns and used return statement, these named returns become redundant. To improve maintainability, the named return for `delegateAllocations` can be removed while a temporary variable for `delegateAllocations` can be used in the corresponding return statement in the following `JBXBuybackDelegate.payParams` function.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171
```solidity
    function payParams(JBPayParamsData calldata _data)
        external
        override
        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
    {
        ...

        // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
            ...

            // Return this delegate as the one to use, and do not mint from the terminal
            delegateAllocations = new JBPayDelegateAllocation[](1);
            delegateAllocations[0] =
                JBPayDelegateAllocation({delegate: IJBPayDelegate(this), amount: _data.amount.value});

            return (0, _data.memo, delegateAllocations);
        }

        // If minting, do not use this as delegate
        return (_data.weight, _data.memo, new JBPayDelegateAllocation[](0));
    }
```

## [11] `control` IS MISTYPED AS `controle`
( Please note that the following instance is not found in https://gist.github.com/itsmetechjay/2efc963de59bcad62e69de48171d10ca#n13-typos. )

In the following comment, `control` is mistyped as `controle`. Please change `controle` to `control`.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L214
```solidity
     * @dev    Slippage controle is achieved here
```

## [12] SOLIDITY VERSION `0.8.20` CAN BE USED
Using the more updated version of Solidity can add new features and enhance security. As described in https://github.com/ethereum/solidity/releases, Version `0.8.20` is the latest version of Solidity, which includes support for Shanghai. To be more secured and more future-proofed, please consider using Version `0.8.20` for the `JBXBuybackDelegate` contract.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2
```solidity
pragma solidity ^0.8.16;
```