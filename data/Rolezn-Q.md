
## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Contracts are not using their OZ Upgradeable counterparts | 2 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Upgrade OpenZeppelin Contract Dependency | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Consider case where `_reservedRate` will exceed `JBConstants.MAX_RESERVED_RATE` which will revert | 1 |
| [LOW&#x2011;4](#LOW&#x2011;4) | `uniswapV3SwapCallback` will revert even though `_amountReceived == _minimumAmountReceived` | 1 |

Total: 5 contexts over 4 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Use Underscores for Number Literals  | 1 |
| [NC&#x2011;2](#NC&#x2011;2) | Use a more recent version of Solidity | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | Empty blocks should be removed or emit something | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Large multiples of ten should use scientific notation | 1 |
| [NC&#x2011;5](#NC&#x2011;5) | Keep using `JBConstants` convention for constants | 1 |


Total: 5 contexts over 5 issues

## Low Risk Issues



### <a href="#QA Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
15: import "@openzeppelin/contracts/access/Ownable.sol";
16: import "@openzeppelin/contracts/interfaces/IERC20.sol";

```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L15-L16



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts



### <a href="#QA Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.8.3 to include critical vulnerability patches.

```solidity
    "@openzeppelin/contracts": "^4.7.3"
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/../package.json#L25



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.3 via `>=4.8.3`



### <a href="#QA Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Consider case where `_reservedRate` will exceed `JBConstants.MAX_RESERVED_RATE` which will revert

Consider the case where `_reservedRate` > `JBConstants.MAX_RESERVED_RATE` which will cause `JBConstants.MAX_RESERVED_RATE - _reservedRate` to underflow and thus revert the `mulDiv` call

#### <ins>Proof Of Concept</ins>

As per below code snippets it is clear that a ticketId claimableAmount can be zero if the time has elapsed for the reward claim. In such case for other valid ticketIds sent in by the user the rewards claim will also fail since the transaction will revert since claimedAmount == 0.

```solidity
        uint256 _nonReservedToken = PRBMath.mulDiv(
            _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
        );
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L279


### <a href="#QA Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> `uniswapV3SwapCallback` will revert even though `_amountReceived == _minimumAmountReceived`

Consider the case where `_reservedRate` > `JBConstants.MAX_RESERVED_RATE` which will cause `JBConstants.MAX_RESERVED_RATE - _reservedRate` to underflow and thus revert the `mulDiv` call

#### <ins>Proof Of Concept</ins>

The function does not validate when `_amountReceived` is equal to `_minimumAmountReceived` which is the minimum amount to receive. In this case, it will revert.

```solidity
if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L228





### <a href="#QA Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Use Underscores for Number Literals

#### <ins>Proof Of Concept</ins>



```solidity
68: uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68









### <a href="#QA Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Use a more recent version of Solidity


<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<a href="https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/">0.8.19</a>:
SMTChecker: New trusted mode that assumes that any compile-time available code is the actual used code, even in external calls. 
Bug Fixes: 
- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in using for.

<a href="https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/">0.8.20</a>:
- Assembler: Use push0 for placing 0 on the stack for EVM versions starting from “Shanghai”. This decreases the deployment and runtime costs.
- EVM: Set default EVM version to “Shanghai”.
- EVM: Support for the EVM Version “Shanghai”.
- Optimizer: Re-implement simplified version of UnusedAssignEliminator and UnusedStoreEliminator. It can correctly remove some unused assignments in deeply nested loops that were ignored by the old version.
- Parser: Unary plus is no longer recognized as a unary operator in the AST and triggers an error at the parsing stage (rather than later during the analysis).
- SMTChecker: Group all messages about unsupported language features in a single warning. The CLI option --model-checker-show-unsupported and the JSON option settings.modelChecker.showUnsupported can be enabled to show the full list.
- SMTChecker: Properties that are proved safe are now reported explicitly at the end of analysis. By default, only the number of safe properties is shown. The CLI option --model-checker-show-proved-safe and the JSON option settings.modelChecker.showProvedSafe can be enabled to show the full list of safe properties.
- Standard JSON Interface: Add experimental support for importing ASTs via Standard JSON.
- Yul EVM Code Transform: If available, use push0 instead of codesize to produce an arbitrary value on stack in order to create equal stack heights between branches.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.16;
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#QA Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Empty blocks should be removed or emit something

#### <ins>Proof Of Concept</ins>

```solidity
239: {}

```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L239






### <a href="#QA Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

#### <ins>Proof Of Concept</ins>



```solidity
68: uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68



### <a href="#QA Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Keep using `JBConstants` convention for constants

It is seen that the protocol uses `JBConstants.sol` to store constants. The contract should keep using the same convention and apply the constant value in `JBConstants.sol`

#### <ins>Proof Of Concept</ins>

You can see the use of `JBConstants` foe example in:

```solidity
279: _amountReceived, JBConstants.MAX_RESERVED_RATE - _reservedRate, JBConstants.MAX_RESERVED_RATE
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L279

This convention should also apply to:
```solidity
68: uint256 private constant SLIPPAGE_DENOMINATOR = 10000;
```

https://github.com/code-423n4/2023-05-juicebox/tree/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68







