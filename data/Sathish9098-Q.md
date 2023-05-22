# LOW FINDINGS

##

## [L-1] _data.beneficiary address not checked before minting tokens. There is possibility to mint token in address(0) 

It's important to ensure that the beneficiary address is properly checked before minting tokens to maintain security and prevent unauthorized minting. Failure to validate the beneficiary address can result in potential security vulnerabilities and unauthorized token creation

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

341:  _beneficiary: _data.beneficiary,

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL341C12-L341C45

##

## [L-2] Missing the sender information in events when Mint or burn Tokens 

Including sender information in events during token minting or burning is a good practice to provide transparency and track the origin of the token creation or destruction. It helps to maintain an audit trail and enables easy verification of token transactions

```solidity
FILE: Breadcrumbs2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

352: emit JBXBuybackDelegate_Mint(_data.projectId);
328: emit JBXBuybackDelegate_Swap(_data.projectId, _data.amount.value, _amountReceived);

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L352

##

## [L-3] abi.decode can throw an exception if the decoding fails due to invalid data or incorrect decoding logic

Proper error handling should be implemented to handle potential exceptions and provide meaningful feedback to the user or handle the failure gracefully within the application

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

153: (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

196:  (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

221:  (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL153C8-L153C115

Recommended Mitigation:

Use try/catch blocks to handle potential exceptions

##

## [L-4] Perform input validation and sanitization before decoding data to mitigate risks

Decoding arbitrary data can introduce security risks, such as potential vulnerabilities from maliciously crafted or manipulated data

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

153: (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

196:  (,, uint256 _quote, uint256 _slippage) = abi.decode(_data.metadata, (bytes32, bytes32, uint256, uint256));

221:  (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL153C8-L153C115

Recommended Mitigation:

Check the data,_data.metadata values before start decoding 

##

## [L-5] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas

```diff
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

- 232: weth.transfer(address(pool), _amountToSend);
+ 232: (bool success, bytes memory data) = address(weth).call(abi.encodeWithSignature("transfer(address,uint256)", address(pool), _amountToSend));
```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL232C9-L232C53

##

## [L-6] Lack of uint256 checks before assigning values to critical parameters 

### amount0Delta,amount1Delta,_amount values not checked before assigning to critical values . There is possibility of human errors 

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

216:   function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external override {

334:  function _mint(JBDidPayData calldata _data, uint256 _amount) internal {

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L216

##

## [L-7] Division before multiplication should be avoided to mitigate unexpected results 

- It is advisable to follow the conventional order of operations, which is performing multiplication before division, to avoid unexpected results and maintain the expected mathematical behavior

- The conventional order of operations, known as PEMDAS (Parentheses, Exponents, Multiplication and Division, and Addition and Subtraction), ensures that calculations are performed in a standardized and predictable manner

```solidity
FILE: 2023-05-juicebox/juice-buyback/contracts/JBXBuybackDelegate.sol

197:  uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);
157:  if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {

```
https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL197C8-L197C97

Recommended Mitigation:

```solidity
_quote - ((_quote * _slippage) / SLIPPAGE_DENOMINATOR);

```












