
## Failure to check address(0) on _data.beneficiary parameter:

### Proof of Concept:

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#LL286C70-L286C70

**Passing address 0 as a parameter can lead to loss of funds. Consider checking it.**