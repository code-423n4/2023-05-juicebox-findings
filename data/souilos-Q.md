# VULN 1 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 232 at contests/2023-05-juice/JBXBuybackDelegate.sol:

        weth.transfer(address(pool), _amountToSend);


Found in line 286 at contests/2023-05-juice/JBXBuybackDelegate.sol:

        if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 2 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 205 at contests/2023-05-juice/JBXBuybackDelegate.sol:

            if (_amountReceived == 0) _mint(_data, _tokenCount);


Found in line 207 at contests/2023-05-juice/JBXBuybackDelegate.sol:

            _mint(_data, _tokenCount);


Found in line 334 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    function _mint(JBDidPayData calldata _data, uint256 _amount) internal {

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 3 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 63 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    bool private immutable _projectTokenIsZero;


Found in line 79 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    IERC20 public immutable projectToken;


Found in line 85 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    IUniswapV3Pool public immutable pool;


Found in line 90 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    IJBPayoutRedemptionPaymentTerminal3_1 public immutable jbxTerminal;


Found in line 95 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    IWETH9 public immutable weth;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.