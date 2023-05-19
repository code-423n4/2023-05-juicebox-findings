## Non Critical Issues

|      | Issue                                                                   |
| ---- | :---------------------------------------------------------------------- |
| NC-1 | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                           |
| NC-2 | FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE                       |
| NC-3 | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION                   |
| NC-4 | NO SAME VALUE INPUT CONTROL                                             |
| NC-5 | ADD PARAMETER TO EVENT-EMIT                                             |
| NC-6 | USE A MORE RECENT VERSION OF SOLIDITY                                   |
| NC-7 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |
| NC-8 | USE UNDERSCORES FOR NUMBER LITERALS                                     |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

54:     event JBXBuybackDelegate_Mint(uint256 projectId);

352:         emit JBXBuybackDelegate_Mint(_data.projectId);

```

### [NC-2] FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

#### Description:

Use camel case for all functions, parameters and variables and snake case for constants.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

197:         uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

```

### [NC-3] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

Use (e.g. 1e5) rather than decimal literals (e.g. 10000), for better code readability.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

68:     uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

```

### [NC-4] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

124:         projectToken = _projectToken;

125:         pool = _pool;

126:         jbxTerminal = _jbxTerminal;

128:         weth = _weth;

```

### [NC-5] ADD PARAMETER TO EVENT-EMIT

#### Description:

Some event-emit description hasn’t parameter. Add to parameter for front-end website or client app , they can has that something has happened on the blockchain.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

352:         emit JBXBuybackDelegate_Mint(_data.projectId);

```

### [NC-6] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

2: pragma solidity ^0.8.16;

```

### [NC-7] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

68:     uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

106:     uint256 private mintedAmount = 1;

113:     uint256 private reservedRate = 1;

334:     function _mint(JBDidPayData calldata _data, uint256 _amount) internal {

```

### [NC-8] USE UNDERSCORES FOR NUMBER LITERALS

#### Recommended Mitigation Steps

Consider using underscores for number literals to improve its readability.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

68:     uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

```

## Low Issues

|     | Issue                                                 |
| --- | :---------------------------------------------------- |
| L-1 | AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS |
| L-2 | Empty Function Body - Consider commenting why         |
| L-3 | LOSS OF PRECISION DUE TO ROUNDING                     |
| L-4 | OWNER CAN RENOUNCE OWNERSHIP                          |
| L-5 | Unsafe ERC20 operation(s)                             |
| L-6 | USE `_SAFEMINT` INSTEAD OF `_MINT`                    |
| L-7 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`              |

### [L-1] AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

232:         weth.transfer(address(pool), _amountToSend);

286:         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-2] Empty Function Body - Consider commenting why

#### Description:

Code contains empty block

Empty blocks should be removed or emit something - The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be `abstract` and the function signatures be added without any default implementation. If the block is an empty `if`-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (`if(x){}else if(y){...}else{...}` => `if(!x){if(y){...}else{...}}`). Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

239:     {}

```

#### Recommended Mitigation Steps:

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### [L-3] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

156:         if (_tokenCount < _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)) {

197:         uint256 _minimumReceivedFromSwap = _quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR);

```

### [L-4] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

39: contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.`

### [L-5] Unsafe ERC20 operation(s)

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

232:         weth.transfer(address(pool), _amountToSend);

286:         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

```

### [L-6] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

205:             if (_amountReceived == 0) _mint(_data, _tokenCount);

207:             _mint(_data, _tokenCount);

334:     function _mint(JBDidPayData calldata _data, uint256 _amount) internal {

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

### [L-7] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

232:         weth.transfer(address(pool), _amountToSend);

286:         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.
