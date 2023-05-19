## Gas Optimizations

|       | Issue                                                                       |
| ----- | :-------------------------------------------------------------------------- |
| GAS-1 | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                  |
| GAS-2 | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE |
| GAS-3 | Setting the constructor to payable                                          |
| GAS-4 | OPTIMIZE NAMES TO SAVE GAS                                                  |
| GAS-5 | USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION      |
| GAS-6 | State variables only set in the constructor should be declared immutable    |
| GAS-7 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                         |
| GAS-8 | USE BYTES32 INSTEAD OF STRING                                               |

### [GAS-1] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

268:             data: abi.encode(_minimumReceivedFromSwap)

```

### [GAS-2] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

232:         weth.transfer(address(pool), _amountToSend);

286:         if (_nonReservedToken != 0) projectToken.transfer(_data.beneficiary, _nonReservedToken);

```

### [GAS-3] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

118:     constructor(

```

### [GAS-4] OPTIMIZE NAMES TO SAVE GAS

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

39: contract JBXBuybackDelegate is IJBFundingCycleDataSource, IJBPayDelegate, IUniswapV3SwapCallback, Ownable {

```

### [GAS-5] USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

2: pragma solidity ^0.8.16;

```

### [GAS-6] State variables only set in the constructor should be declared immutable

#### Description:

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

118:     constructor(
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

### [GAS-7] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

205:    if (_amountReceived == 0) _mint(_data, _tokenCount);
        } else {
            _mint(_data, _tokenCount);
        }

```

### [GAS-8] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

147:         returns (uint256 weight, string memory memo, JBPayDelegateAllocation[] memory delegateAllocations)

238:         returns (uint256 reclaimAmount, string memory memo, JBRedemptionDelegateAllocation[] memory delegateAllocations)

```
