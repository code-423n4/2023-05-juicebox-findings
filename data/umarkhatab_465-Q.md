## [N‑01] The Non-library/interface files should not use floating compiler versions but a specific compiler.
Have a look here 

### Description:

Currently, the contract uses the floating versions of the compiler

```solidity
pragma solidity ^0.8.16;

```

that has to be changed to 

```solidity
pragma solidity 0.8.16;

```