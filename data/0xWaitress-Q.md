1. payParams can be view functions

there is no write operation in payParams; only passing of calldata and checks on price to return either a set of data for swapping or minting.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L144-L171

```solidity
    function payParams(JBPayParamsData calldata _data)
        external
        override
        returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)
    {
        // Find the total number of tokens to mint, as a fixed point number with 18 decimals
        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

        // Unpack the quote from the pool, given by the frontend
        (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

        // If the amount swapped is bigger than the lowest received when minting, use the swap pathway
        if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {
            // Pass the quote and reserve rate via a mutex
            mintedAmount = _tokenCount;
            reservedRate = _data.reservedRate;

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