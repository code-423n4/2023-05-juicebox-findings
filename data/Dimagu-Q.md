# C4 Audit Juicebox JBXBuybackDelegate (May 22, 2023)

## QA Report (with low) Submission

### Low Risk

#### [L-1] Check \_slippage is in specified range and \_quote is a valid quote `JBXBuybackDelegate.payParams` in [line 153](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL153-LL154)

Currently, `uint256 _slippage` and `_quote`variable is not sanitized, and an arbitrary value could be assigned to it through `_data.metadata`.

This could lead to the follwoing unexpected behavior:

1. `_slippage` to be greater than `SLIPPAGE_DENOMINATOR`, which will lead to `_slippage / SLIPPAGE_DENOMINATOR > 1`, and the equation:
   `_quote - (_quote * _slippage / SLIPPAGE_DENOMINATOR)` will cause an arithmetic underflow
2. `_slipage `can be arbitrarly set to 0, which might lead to miscalculations of either the protocol should mint or swap tokens
3. It is a better idea to fetch the `_quote` for the token pair inside `JBXBuybackDelegate.payParams` instead of passing it through `jbEthPaymentTerminal.pay()` function through the `_metadata`. Currently, arbitrary value can be passed for `_quote` which might lead to to miscalculations of either the protocol should mint or swap tokens.

#### [L-2] Function `payParams(JBPayParamsData calldata \_data)' in [line 144](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL144-LL171) should be protected with an access control

As anyone can call this function, a malicious user can set the value of the 2 mutex `_mintedAmount and _reservedRate`to 0, making every call much more expensive (cold variable access)

### Quality Assurance (QA)

#### [QA-1] Unnecessary Tuple for decoding one value

`(uint256 \_minimumAmountReceived)`in [line 221](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL221) is presented as a tuple, but it is a single value. The parenthesis are not necessary.

#### [QA-2] Compilation breaks with solidity versions `0.8.16/17/18`

`UniswapV3ForgeQuoter.sol`referenced in package.json [line 13](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/package.json#LL13) is written in solidity compiler version `0.8.19`, which causes a compilation in solidity versions `0.8.16/17/18` to fail.

#### [QA-3] Unnecessary imports and inherited contracts

1. `import "@jbx-protocol/juice-contracts-v3/contracts/interfaces/IJBFundingCycleBallot.sol";` is not needed, [line 8](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL8)
2. `import "@openzeppelin/contracts/access/Ownable.sol";` is not needed, [line 15](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL15)
3. There is no need for `JBXBuybackDelegate` contract to inherit `Ownable` contract, [line 39](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL39)

#### [QA-4] Underscore in constant assignment missing, [line 68](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL68)

`SLIPPAGE_DENOMINATOR = 10000` should be changed to `SLIPPAGE_DENOMINATOR = 10_000` for improved readability.

#### [QA-5] Typos in comments:

1. [Line 32](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL32): `weigh` should be changed to `weight`
2. [Line 102](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL102) & [199](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL199): typo `prefered` should be changed to `preferred`
3. [Line 187](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL187): `Retrieve the number of token created` should be changed to `Retrieve the number of tokens created`
4. [Line 204](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL204): `If swap failed, mint instead, with the original weight + add to balance the token in` should be changed to `If swap failed, mint instead with the original weight + add to balance the token in`
5. [Line 212](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL212): `(where token transfer should happens)` should be changed to `(where token transfer should happen)`
6. [Line 214](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL214): `Slippage controle is achieved here` should be changed to `Slippage control is achieved here`
7. [Line 246](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL246): `Swap the terminal token to receive the project toke_beforeTransferTon` incomprehensible comment
8. [Line 248](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL248): `This delegate first receive the whole amount of project token` should be changed to `This delegate first receives the whole amount of project tokens`
9. [Line 250](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL250): `then burn the rest of this delegate balance (ie the amount of reserved token)` should be changed to `then burn the rest of this delegate balance (ie the amount of reserved tokens)`
10. [Line 288](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL288): `If there are reserved token...` should be changed to `If there are reserved tokens...`
11. [Line 292](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL292): `Burn all the reserved token...` should be changed to `Burn all the reserved tokens...`
12. [Line 301](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL301): `Mint the reserved token with...` should be changed to `Mint the reserved tokens with...`
13. [Line 311](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL311) `Burn the non-reserve token...` should be changed to `Burn the non-reserve tokens...`
14. [Line 332](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL322): `the amount of token out to mint` should be changed to `the amount of tokens out to mint`

#### [QA-6] Keep consistent specific naming for the comments/variables/functions:

1. [Line 53](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL53) rename the 3rd event parameter from `amountOut` to `amountJBX`
2. [Line 84](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL84) instead of `projectToken` use `JBXToken`
3. [Line 87](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL87) instead of `The uniswap pool corresponding to the project token - other token market` use `The uniswap pool corresponding to the JBX token - wETH market`
4. [Line 95](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL95) instead of `jbxTerminal` use `jbEthPaymentTerminal`

#### [QA-7] Use more generalised titles:

1. [Line 57](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL57) instead of `private constant properties` use `private constant/immutable properties`
2. [Line 76](https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#LL76) instead of `public constant properties` use `public constant/immutable properties`