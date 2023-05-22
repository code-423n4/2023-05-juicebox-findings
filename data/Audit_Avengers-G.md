## uint256 is too big for certain variables
heyy team, I hope that you are all doing well, basically uint256 is too large for certain variables like
_tokenCount and _reservedRate because they aren't inside a loop and aren't being incremented or something, the only that is somewhat close is the swapping thing but even then uint256 too huge.
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L188
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L192

Thank you have a nice day.