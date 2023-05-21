# QA Report for contest Juicebox Buyback Delegate

## Overview
During the audit, 1 low, 2 refactoring and 5 non-critical issues were found.

### Low Risk Issues

Total: 1 instance over 1 issues

|#|Issue|Instances|
|-|:-|:-:|
| [L-01] | `JBXBuybackDelegate` deployer should decide if held fees should be refunded | 1 |

### Refactoring Issues

Total: 2 instances over 2 issues

|#|Issue|Instances|
|-|:-|:-:|
| [R-01]| Better and documentation naming for `_projectTokenIsZero`  | 1 |
| [R-02]| Use named function calls | 1 |

### Non-critical Issues

Total: 13 instances over 5 issues

|#|Issue|Instances|
|-|:-|:-:|
| [NC-01]| Use `JBTokenAmount::decimals` instead of hardcoding it in `payParams` | 1 |
| [NC-02]| Memo is not passed in all cases | 3 |
| [NC-03]| Missing, misleading, incorrect, or incomplete comments | 7 |
| [NC-04]| Events missing key information | 1 |
| [NC-05]| Empty methods should be thoroughly documented | 1 |

#
## Low Risk Issues (1)
#

### [L-01] `JBXBuybackDelegate` deployer should decide if held fees should be refunded
##### Description

When minting of tokens is done, via `JBXBuybackDelegate::_mint`, ETH is sent back to the terminal via a call to `addToBalanceOf`.

There are 2 versions of `addToBalanceOf`. One that [accepts a flag `_shouldRefundHeldFees` indicating if held fees should be refunded based on the amount being added](https://github.com/jbx-protocol/juice-contracts-v3/blob/main/contracts/abstract/JBPayoutRedemptionPaymentTerminal3_1.sol#L686-L696).

and other, with one less argument, that is used by our protocol that [sets it to false by default](https://github.com/jbx-protocol/juice-contracts-v3/blob/main/contracts/abstract/JBPayoutRedemptionPaymentTerminal3_1.sol#L559-L578).

This option should not be hardcoded as it is, a contract deployer should determine if he opts in or not.

##### Instances (1)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L347-L350

##### Recommendation

Add a constructor parameter that is then used to with the `addToBalanceOf` when `_mint` is called.

#

#### Refactoring Issues (2)
#

### [R-01] Better and documentation naming for `_projectTokenIsZero` 
##### Description

A variable named `_projectTokenIsZero` is used to indicate if the project token address is less then the address terminal token by sort order.
It describes it simply as:
```
    /**
     * @notice Address project token < address terminal token ?
     */
    bool private immutable _projectTokenIsZero;
```
then it is used to identify which swap return amount is to be used.
```Solidity
            _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));
```

This is needed because Uniswap `IUniswapV3Pool::pool` returns the token amounts corresponding to the token sorted addresses order. 
This is implicit because pools are created with the tokens as such ([IUniswapV3Factory#PoolCreated](https://docs.uniswap.org/contracts/v3/reference/core/interfaces/IUniswapV3Factory#poolcreated))

The naming and description can be changed to better describe the need for this flag variable.

##### Instances (1)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L60-L63

##### Recommendation

Add a more descriptive name and documentation. Example:
```
    /**
     * @notice sores if: address project token < address terminal token
     * @dev used to identify which pair token (0 or 1) is the project token in a swap result
     */
    bool private immutable _projectTokenIsPairToken0;
```

#

### [R-02] Use named function calls
##### Description

Code base has an extensive use of named function calls, but it somehow missed one instance where this would be appropriate.

```Solidity
        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );
```

([addToBalanceOf](https://github.com/jbx-protocol/juice-contracts-v3/blob/main/contracts/abstract/JBPayoutRedemptionPaymentTerminal3_1.sol#L569-L574))

##### Instances (1)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L348-L350

##### Recommendation

Use named function calls on function call, as such:
```Solidity
    jbxTerminal.addToBalanceOf{value: _data.amount.value}({
        _projectId: _data.projectId,
        _amount: _data.amount.value,
        _token: JBTokens.ETH,
        _memo: "",
        _metadata: new bytes(0)
    });
```

#

## Non-critical Issues (5)
#

### [NC-01] Use `JBTokenAmount::decimals` instead of hardcoding it in `payParams`
##### Description

In `payParams` the number of tokens to mint is determined by multiplying value with weight and dividing by *1e18*.

```Solidity
        // Find the total number of tokens to mint, as a fixed point number with 18 decimals
        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);
```

As the token to be use by the project has 18 decimals this is not an issue for now. Any other project token to be added will require this modified in order to keep the
premise that the total number of tokens to mint is a fixed point number with 18 decimals.

Out of the formula, `_data.weight` has 18 decimals

https://github.com/jbx-protocol/juice-contracts-v3/blob/12d852f28d372dd44987586f8009c56b0fe247a9/contracts/JBSingleTokenPaymentTerminalStore3_1.sol#L351-L352
```Solidity
    // The weight according to which new token supply is to be minted, as a fixed point number with 18 decimals.
    uint256 _weight;
```

so the the 1e18 must be the decimal count of the `JBTokenAmount` in order to be fully safe.

##### Instances (1)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L149-L150

##### Recommendation

Use the decimals propriety of `JBTokenAmount` instead of a hardcoded *1e18*:
```Solidity
        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** _data.amount.decimals);
```

#

### [NC-02] Memo is not passed in all cases
##### Description

All controller operations of `mintTokensOf` and `burnTokensOf` or terminal operation of `addToBalanceOf` have a `_memo` parameter. 
The information in the memo will be passed to an event. The event is sent regardless so adding the memo will make off-chain analytics clearer.

##### Instances (3)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L297
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L319
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L349 

##### Recommendation

Pass the memo information to the mentioned calls

#

### [NC-03] Missing, misleading, incorrect, or incomplete comments
##### Description
There are cases where the comments are either missing, incomplete or incorrect throughout the codebase.

##### Instances (7)

- `_swap`: 
    - at line 246 *toke_beforeTransferTon* should be *token_beforeTransferToken* ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L246))
    - at line 251 *fc* should be properly described as *funding cycle* ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L251))
    - at line 301 the comments after `result: ` are ambiguous `_amountReceived-reserved here, reservedToken in reserve`, consider making them more clear, example: `(_amountReceived - reserved) here, (reserved (token)) in reserve` ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L301))
    - at lines 250 and 252 *ie* should be *i.e.* ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L250-L252))
- `_mint`: at line 329 there is a double *in in*, remove one *in* ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L329))
- `uniswapV3SwapCallback`: 
    - at line 214 typo in `controle`, an extra *e*, remove the extra *e* ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L214))
    - at line 212 typo, "should happen" or rephrase to "happens" ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L212))

##### Recommendation

Resolve the mentioned issues.

#

### [NC-04] Events missing key information
##### Description

Some events are missing key information when emitted.

##### Instances (1)

- in `_mint` the `JBXBuybackDelegate_Mint` event should contain the minted amount and beneficiary to whom it was minted ([code](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L352))

##### Recommendation

Add the missing information.

#

### [NC-05] Empty methods should be thoroughly documented
##### Description

The function `redeemParams` variable has no code implementation. 
It is both lacking any NatSpec and any internal comments to indicate what is its purpose.

##### Instances (1)

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L235-L239

##### Recommendation

Properly document the method, why it is needed and why it is empty.

#
