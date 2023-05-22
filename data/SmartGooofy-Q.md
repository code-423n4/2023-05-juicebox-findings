# ownable 

The contract inherits from Ownable, however nowhere in the code is the owner used. Therefore, remove it from the code.

# uniswapV3SwapCallback function

Line 217

Introduce a *require* statement to validate if any tokens have been swapped. If no swap has occurred, the operation should be immediately reverted to prevent needless execution of the code.

I suggest to modify the function as follows:

      function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {
        require(amount0Delta !=0 && amount1Delta != 0, "Nothing to swap");
        ...

# _swap function

Line 262

Verify the value of *_reservedRate*. It must not exceed *JBConstants.MAX_RESERVED_RATE*. If this condition is not met, the function should terminate and revert the transaction.

    require(_reservedRate <= JBConstants.MAX_RESERVED_RATE, "Invalid value for reserve rate" ); 

# superfluous variable

Line 312

It appears the variable *_nonReservedTokenInContract* and its computation are unnecessary. On line 283, *_reservedToken* is derived from *_amountReceived* and *_nonReservedToken*. Since neither *_reservedToken* nor *_amountReceived* are altered by the time line 312 is reached, *_nonReservedTokenInContract* is bound to hold the same value as *_nonReservedToken*. Consequently, line 312 can be omitted and *_nonReservedTokenInContract* can be substituted with *_nonReservedToken* elsewhere in the code. 

    if (_nonReservedToken != 0) {
                controller.burnTokensOf({
                    _holder: address(this),
                    _projectId: _data.projectId,
                    _tokenCount: _nonReservedToken,
                    _memo: "",
                    _preferClaimedTokens: false
                });
            }

# Unverified return value

Line 232

The transfer function of WETH is invoked, yet its return value is not being verified. I recommend validating the return value and executing a revert operation if the transfer is unsuccessful.

    bool success = weth.transfer(address(pool), _amountToSend);
    require(success, "Transfer failed.");
 
# _swap function has a lot of code lines

Adhering to best practices, it's beneficial to avoid overextending function lengths as this enhances readability. I recommend extracting the segment of code responsible for adding the reserved tokens to the reserve and encapsulating it within a separate internal function. This newly created function could then be invoked at line 293.

    function reserveTokens(...) internal{
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
