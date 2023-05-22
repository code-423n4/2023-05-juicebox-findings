## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | For immutable variables, Zero address checks are missing in constructor | 4 |

### Non-Critical Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [N&#x2011;01] | Use latest version of PRBMath library | 1 |
| [N&#x2011;02] | Use a more recent version of Solidity | 1 |

### Low Risk Issues
### [L&#x2011;01]  For immutable variables, Zero address checks are missing in constructor
Zero address check validations should be used in the constructors, to avoid the risk of setting a immutable storage variable as zero address at the time of deployment. If bymistake, address(0) is set it will cause redeployment of contract.

There are 4 instances of this issue:

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

118    constructor(
119        IERC20 _projectToken,
120        IWETH9 _weth,
121        IUniswapV3Pool _pool,
122        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
123    ) {
124        projectToken = _projectToken;
125        pool = _pool;
126        jbxTerminal = _jbxTerminal;
127        _projectTokenIsZero = address(_projectToken) < address(_weth);
128        weth = _weth;
129    }
```
[Link to code](https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L118-L129)

### Recommended Mitigation steps
Add address(0) validation check in constructor.

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

    constructor(
        IERC20 _projectToken,
        IWETH9 _weth,
        IUniswapV3Pool _pool,
        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
    ) {
+       require(address(_projectToken) != address(0), "invalid address");
+       require(address(_weth) != address(0), "invalid address");
+       require(address(_pool) != address(0), "invalid address");
+       require(address(_jbxTerminal) != address(0), "invalid address");
        projectToken = _projectToken;
        pool = _pool;
        jbxTerminal = _jbxTerminal;
        _projectTokenIsZero = address(_projectToken) < address(_weth);
        weth = _weth;
    }
```

### Non-Critical Issues
### [N&#x2011;01]  Use latest version of PRBMath library
The contract uses old version of PRBMath library 3.7.0. The latest version v4.0.0 has lots of fixes and some breaking changes. It is recommended to use latest version of PRBMath library.

[Link to latest release features](https://github.com/PaulRBerg/prb-math/releases/tag/v4.0.0)

There is 1 instance of this issue.

```solidity
File: /juice-buyback/package.json

 "@paulrberg/contracts": "^3.7.0",
```

### [N&#x2011;02]  Use a more recent version of Solidity
For security and optimization, it is best practice to use the latest Solidity version.
For the security fix list in the versions: [Link to reference](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

There is 1 instance of this issue.

```solidity
File: juice-buyback/contracts/JBXBuybackDelegate.sol

pragma solidity ^0.8.16;
```