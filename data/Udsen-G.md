## 1. STATE VARIABLES SHOULD BE CACHED TO AVOID SECOND ACCESS OF IT WITHIN THE FUNCTION TO SAVE ONE `SSTORE` OPERATION.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224

Similarly in the `JBXBuybackDelegate.uniswapV3SwapCallback()` function there is a second access of the `pool` state variable. This variable can be cached and memory variable can be used instead to save gas.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L218
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L232

Similarly in the `JBXBuybackDelegate._mint()` function, the `jbxTerminal` state variable is used for the second time. This variable holds the address of the `IJBController` interface. Hence this can be cached into a local memory variable and used from there.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L335
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L348

## 2. FUNCTIONS WITHOUT IMPLEMENTATION CAN BE REMOVED TO SAVE DEPLOYMENT GAS COST.

The `JBXBuybackDelegate.redeemParams()` function implementation is missing. Hence this can be removed from the bytecode to reduce the deployment gas cost.

    function redeemParams(JBRedeemParamsData calldata _data)
        external
        override
        returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)
    {}


https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

## 3. REDUNDANT ARITHMETIC OPERATION CAN BE REMOVED TO SAVE DEPLOYMENT AND RUNTIME GAS COST

In the `JBXBuybackDelegate._swap()` function, the `_nonReservedTokenInContract` is calculated as follows:

            uint256 _nonReservedTokenInContract = _amountReceived - _reservedToken;

In one of the earlier operations in the `JBXBuybackDelegate._swap()` function, following arithmetic operation is performed.

            uint256 _reservedToken = _amountReceived - _nonReservedToken;

This means :

            _nonReservedToken = _amountReceived - _reservedToken;

Hence there is no need to perform the arithmetic operation to calculate `_nonReservedTokenInContract`. Since `_nonReservedToken` variable can be directly used inplace of the `_nonReservedTokenInContract` variable. Hence this will save gas since no need to perform the assignment and substraction operation.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L283
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L312