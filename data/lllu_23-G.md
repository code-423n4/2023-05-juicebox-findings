# [G-01] Using `bools` for storage incurs overhead
## `Booleans` are more expensive than uint256 or any type that takes up a full  word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

References: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

There are 1 instances of this issue:

`File: contracts/JBXBuybackDelegate.sol`
`63:    bool private immutable _projectTokenIsZero;`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L63

# [G-02]Use hardcode address instead `address(this)`
## Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded`address`. Foundry’s `script.sol` and solmate’s `LibRlp.sol` contracts can help achieve this.

### References:
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

There are 2 instances of this issue:

`File:  contracts/mock/MockAllocator.sol`
`26:          JBTokenAmount(address(this), 1 ether, 10 ** 18, 0),`
`27:          JBTokenAmount(address(this), 1 ether, 10 ** 18, 0),`
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/mock/MockAllocator.sol#L26-#L27

# [G-03]Optimize names to save gas
## Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more here.
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Proof Of Concept
All in-scope contracts.

Recommended Mitigation Steps
Find a lower method ID name for the most called functions for example `Call()` vs .`Call1()` is cheaper by   `22` gas.

For example, the function IDs in the `Gauge.sol` contract will be the most used; A lower method ID may be given.

# [G-04] Use solidity version ` 0.8.19` to gain some gas boost
## Upgrade to the latest solidity version `0.8.19` to get additional gas savings.

See latest release for reference:
https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement