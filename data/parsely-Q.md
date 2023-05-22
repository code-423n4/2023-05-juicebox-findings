# [L-01] In the _mint function the call to `jbxTerminal.addToBalanceOf` does not check the value that was send or the current balance of the contract before sending ETH to the terminal, if there is too little balance this could cause a revert.
## Context
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L348-L353
```
        // Send the eth back to the terminal balance
        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );

        emit JBXBuybackDelegate_Mint(_data.projectId);
    }
```
## Description
The call to the _mint function happens in the `didPay()` function, the balance of the contract or the current balance of the contract is not checked before the ETH is sent to the terminal, without context of previous calls, if there is any possibility of the accounting being wrong, a legitimate call could revert. Although the call is also future proofing to cater for future ERC20 token terminals , it would be a good idea to check the accounting before the call.

## Resolution
It could be considered to check the value that was received from the `terminal.pay()` call and also the current contract balance, for either ETH or ERC20 tokens.

# [L-02] In the _mint function the amount of tokens returned by the `controller.mintTokensOf` is not checked.
## Context
https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L338-L345
```
        // Mint to the beneficiary with the fc reserve rate
        controller.mintTokensOf({
            _projectId: _data.projectId,
            _tokenCount: _amount,
            _beneficiary: _data.beneficiary,
            _memo: _data.memo,
            _preferClaimedTokens: _data.preferClaimedTokens,
            _useReservedRate: true
        });
```
## Description
The call to the _mint function happens in the `didPay()` function, the call to `controller.mintTokensOf` does return the resulting number of minted tokens, and it could be considered to check that they are within expected range and that an error has not occurred.

## Resolution
Implement a check to confirm the resulting number of tokens from the call to `mintTokensOf`.