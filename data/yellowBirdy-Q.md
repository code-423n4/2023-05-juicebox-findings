`_swap` calculates the unreserved amount relying on `JBConstants.MAX_RESERVED_RATE`. It's a parameter which could change in future version of the protocol leading to unexpected values (likely too high reserved value)

https://github.com/code-423n4/2023-05-juicebox/blob/9d0458282511ff269b3b35b5b082b56d5cc08663/juice-buyback/contracts/JBXBuybackDelegate.sol#L278-L280

In order to mitigate another constant should be added and used e.g. `JBConstants.RESERVED_RATE_MANTISA`