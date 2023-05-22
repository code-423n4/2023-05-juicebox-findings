# 1: LOCK PRAGMAS TO SPECIFIC COMPILER VERSION

Vulnerability details

## Context:

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case of contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

For reference, see https://swcregistry.io/docs/SWC-103

## Proof of Concept

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler versions.

For reference, see https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/
