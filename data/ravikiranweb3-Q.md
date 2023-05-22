a) On minting the token, JBXBuybackDelegate_Mint() event is emitted with project id only. It would be more appropriate to emit the amount if not beneficiary incase offchain was tracking the minting amounts.


b) function redeemParams is not implemented. It is a dummy implementation.

c) Memo field value of JBDidPayData calldata could have been used for the memo field to retain the context of the transaction instead of passing an empty string.

https://github.com/code-423n4/2023-05-juicebox/blob/main/juice-buyback/contracts/JBXBuybackDelegate.sol#L348-L350

        jbxTerminal.addToBalanceOf{value: _data.amount.value}(
            _data.projectId, _data.amount.value, JBTokens.ETH, "", new bytes(0)
        );

        // @audit -> could have passed _data.memo instead of the empty string