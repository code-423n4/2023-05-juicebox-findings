## [N-01] Use a modifier for access control

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

There are 2 instances of this issue in 2 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    185: if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

    218: if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L185

## [N-02] Contract files should define a fixed compiler version

Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    2: pragma solidity ^0.8.16;

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2

## [N-03] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    150: uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L150

## [N-04] Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    68: uint256 private constant SLIPPAGE_DENOMINATOR = 10000;

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L68

## [N-05] Take advantage of Custom Error return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

There are 3 instances of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    185: if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();

    213: if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();

    228: if (_amountReceived < _minimumAmountReceived) revert JuiceBuyback_MaximumSlippage();

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L185

## [N-06] Using Vulnerable Version of OpenZeppelin

The package.json configuration file says that the project is using 4.7.3 of OpenZeppelin which has a vulnerability.

Although there is no security vulnerability covering the project, it is recommended to use the latest version 4.8.3.

There is 1 instance of this issue in 1 file:

    File: juice-buyback/package.json

    25: "@openzeppelin/contracts": "^4.7.3",

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L25

## [N-07] Use a more recent version of solidity

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    2: pragma solidity ^0.8.16;

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L2