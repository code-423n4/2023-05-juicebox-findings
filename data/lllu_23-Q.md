# [N‑01] Use scientific notation `(e.g. 1e18)` rather than exponentiation `(e.g. 10**18)`
## While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

There are 2 instances of this issue:

`File:  contracts/mock/MockAllocator.sol`
`26:          JBTokenAmount(address(this), 1 ether, 10 ** 18, 0),`
`27:          JBTokenAmount(address(this), 1 ether, 10 ** 18, 0),`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/mock/MockAllocator.sol#L26-#L27

`150:        uint256 _tokenCount = PRBMath.mulDiv(_data.amount.value, _data.weight, 10 ** 18);`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L150