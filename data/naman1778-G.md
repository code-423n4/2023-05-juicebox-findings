## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

Consider using abi.encodePacked() here:

use abi.encodePacked() where possible to save gas

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    268: data: abi.encode(_minimumReceivedFromSwap)

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

    | Contract0 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 101871                                    | 683             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |
    
    
    | Contract1 contract                        |                 |      |        |      |         |
    |-------------------------------------------|-----------------|------|--------|------|---------|
    | Deployment Cost                           | Deployment Size |      |        |      |         |
    | 99465                                     | 671             |      |        |      |         |
    | Function Name                             | min             | avg  | median | max  | # calls |
    | optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

There are 3 instances of this issue in 1 files:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    218: if (msg.sender != address(pool)) revert JuiceBuyback_Unauthorized();
    232: weth.transfer(address(pool), _amountToSend);

    224: uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
    225: uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

    265: zeroForOne: !_projectTokenIsZero,
    267: sqrtPriceLimitX96: _projectTokenIsZero ? TickMath.MAX_SQRT_RATIO - 1 : TickMath.MIN_SQRT_RATIO + 1,
    271: _amountReceived = uint256(-(_projectTokenIsZero ? amount0 : amount1));

#### Test Code

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.not_optimized();
        c1.optimized();
    }
}

contract Contract0 {
     uint8 num=0;
     function not_optimized() public returns(uint8){
        if(num==0)
        {
           return num;
        }
     }
}

contract Contract1 {
     uint8 num=0;
     function optimized() public returns(uint8){
        uint8 num1 = num;
        if(num1==0)
        {
           return num1;
        }
     }
}

#### Gas Report 

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 32299                                     | 190             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2415            | 2415 | 2415   | 2415 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 31699                                     | 187             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2312            | 2312 | 2312   | 2312 | 1       |

## [G-03] Using XOR (^) and AND (&) bitwise equivalents

There is 1 instance of this issue in 1 file:

    File: juice-buyback/contracts/JBXBuybackDelegate.sol

    360: return _interfaceId == type(IJBFundingCycleDataSource).interfaceId
    361:     || _interfaceId == type(IJBPayDelegate).interfaceId;

#### Test Code

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.not_optimized(1,2);
        c1.optimized(1,2);
    }
}

contract Contract0 {
    function not_optimized(uint8 a,uint8 b) public returns(bool){
        return ((a==1) || (b==1));
    }
}

contract Contract1 {
    function optimized(uint8 a,uint8 b) public returns(bool){
        return ((a ^ 1) & (b ^ 1)) == 0;
    }
}

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |