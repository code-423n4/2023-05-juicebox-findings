# Gas Optimizations


### [G&#x2011;01] Declared variables for `mintedAmount` and `reservedRate` state variables can be avoided and mutex reset could be done after sucessful `_swap()` and `_mint()` methods- SAVE 85 gas per didPay() call

Both these state variables are accessed once, gas can be saved by using them directly.
#### In Current Logic  
`mintedAmount` will be used once in case of (`preferClaimedTokens == true` and `_amountReceived == 0`) or 
(`preferClaimedTokens == false`)

`reservedRate` will be used once if (`preferClaimedTokens == true`) while in the case of (`preferClaimedTokens == false`) it is not used once too.
Further without confirmation of successful `_swap()` and `_mint()` resetting the state variable from storage to 1 is itself a big factor regarding high gas, here gas can be saved by resetting them to 1 after the `_swap()` and -`_mint()` as they are not making affect to these internal methods  
didPay() consumption of gas
min- 51675, avg - 161126, median- 148266 and max 242106
#### Suggested Logic
So using these variables directly and reseting the `mutex` after `_swap()` and `_mint()` will make more sense.
In this case `first` we will be saving gas by not declaring variable for both state variables and `second` if `preferClaimedTokens= false` there won't be any read of `reservedRate` state variables
```solidity
 function didPay(JBDidPayData calldata _data) external payable override {
    // Access control as minting is authorized to this delegate
    if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

    // Retrieve the number of token created if minting and reset the mutex (not exposed in JBDidPayData)
    // uint256 _tokenCount = mintedAmount;
    // mintedAmount = 1;

    // Retrieve the fc reserved rate and reset the mutex
    // uint256 _reservedRate = reservedRate;
    // reservedRate = 1;

    // The minimum amount of token received if swapping
    (, , uint256 _quote, uint256 _slippage) = abi.decode(
      _data.metadata,
      (bytes32, bytes32, uint256, uint256)
    );
    uint256 _minimumReceivedFromSwap = _quote - ((_quote * _slippage) / SLIPPAGE_DENOMINATOR);

    // Pick the appropriate pathway (swap vs mint), use mint if non-claimed prefered
    if (_data.preferClaimedTokens) {
      // Try swapping
      uint256 _amountReceived = _swap(_data, _minimumReceivedFromSwap, reservedRate);

      // If swap failed, mint instead, with the original weight + add to balance the token in
      if (_amountReceived == 0) _mint(_data, mintedAmount);
    } else {
      _mint(_data, mintedAmount);
    }

    mintedAmount = 1;
    reservedRate=1;
  }
```
didPay() consumption of gas after making these changes
min- 51580, avg - 161039, median- 148177 and max 242011

by this we can save avg 85 gas per didPay() call


*There is two instance of this issue:*

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol
188:         uint256 _tokenCount = mintedAmount;
192:         uint256 _reservedRate = reservedRate;
```
https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L188
https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L192

