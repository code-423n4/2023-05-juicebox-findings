# VULN 1 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 264 at contests/2023-05-juice/JBXBuybackDelegate.sol:

            recipient: address(this),


Found in line 294 at contests/2023-05-juice/JBXBuybackDelegate.sol:

                _holder: address(this),


Found in line 305 at contests/2023-05-juice/JBXBuybackDelegate.sol:

                _beneficiary: address(this),


Found in line 316 at contests/2023-05-juice/JBXBuybackDelegate.sol:

                    _holder: address(this),

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 2 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 239 at contests/2023-05-juice/JBXBuybackDelegate.sol:

    {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).