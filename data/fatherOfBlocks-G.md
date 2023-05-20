- L312 - The operation _amountReceived - _reservedToken could be unchecked since _reservedToken = _amountReceived - "something" therefore it will never generate underflow.

- L197 - The variable in memory _minimumReceivedFromSwap is created, but it is only used within the if of line 200, therefore gas would be saved, if only that operation were carried out within the if, if it were not executed.

- L202/205 - It is not necessary to create a variable in memory if it is only going to be used once and does not provide further insight.

