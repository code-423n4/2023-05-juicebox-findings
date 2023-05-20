[L-01] Unsafe Casting between types without overflow checks
Apart from the unsafe casting reported by the bot, there are other instances of the same error on the contract.

- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L266
- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L271
- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L224
- https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L225

**Fix:**
- The recommendation is to use OpenZeppelin’s SafeCast library which provides overflow checking when casting from one type of number to another. Consider using SafeCast when casting between types to prevent overflow errors.

  - As reported by OpenZeppelin on the **[Notional Audit](https://blog.openzeppelin.com/notional-audit/#phase-1) ([M02] Casting between types without overflow checks)**, casting between uint to int and viceversa carries the potential of overflow errors