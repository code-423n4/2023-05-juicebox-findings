# Incorrect authorization check in the 'didPay' function
The bug in the provided code is related to an incorrect authorization check in the didPay function. The original code uses the revert statement to handle unauthorized access, but it should be using the require statement instead.

In the incorrect code:
<https://github.com/code-423n4/2023-05-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L185>
```
if (msg.sender != address(jbxTerminal)) revert JuiceBuyback_Unauthorized();
```

The condition checks if the msg.sender (the caller of the function) is not equal to the address(jbxTerminal). If the condition evaluates to true, it triggers a revert statement, which reverts the transaction and rolls back any changes made.

However, using revert in this context is not the appropriate way to handle authorization checks. The revert statement is typically used to revert the entire transaction when a condition is not met. In this case, it should be using the require statement, which is specifically designed for authorization checks.

The correct code should be:

```
require(msg.sender == address(jbxTerminal), "JuiceBuyback_Unauthorized");
```
The require statement checks if the condition ```(msg.sender == address(jbxTerminal))``` is true. If the condition is false, it will revert the transaction and provide an error message ("JuiceBuyback_Unauthorized").

Using require ensures that the condition for authorization is met, and if not, it immediately halts the execution and reverts the transaction, preventing unauthorized access to the didPay function.

By replacing revert with require, the code enforces proper authorization checks and enhances the security of the smart contract.