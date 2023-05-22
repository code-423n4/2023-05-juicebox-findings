1) non checked return of "mintTokensOf" and require() that to not be 0:
```
require(controller.mintTokensOf({
                _projectId: _data.projectId,
                _tokenCount: _amountReceived,
                _beneficiary: address(this),
                _memo: _data.memo,
                _preferClaimedTokens: false,
                _useReservedRate: true
            })!=0,"wanna mint 0 tokens?!");
```