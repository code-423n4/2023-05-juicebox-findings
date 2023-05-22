[L-01]: Use the secure transfer helper library for ERC20 transfer.
1. The _swap#transfer function does not check the return value of these calls. Tokens that return false rather than being returned to indicate a failed transfer may silently fail and not be returned as expected.
2. Because the IERC20 interface requires a boolean return value, an ERC20 transmission attempt with missing return values will be aborted. This means that Juicebox payment terminals cannot support a number of popular ERC20s, including USDT and BNB.
Recommended mitigation steps:
Use a secure transfer library such as OpenZeppelin's SafeERC20 to ensure consistent handling of ERC20 return values and abstract away inconsistent ERC20 implementations.

[L-02]: Use the latest up-to-date version of Solidity.