################################################################################
Contracts with signature
################################################################################

Since the language of contracts writing allows performing enclosed contracts, it is possible to fulfill such an enclosed contract without the knowledge of the user who has run the external contract that may lead to the user's signature of transactions unauthorized by it, let's say the transfer of money from its account.

Let's suppose there is a MoneyTransfer Contract *MoneyTransfer*:

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

If in a contract launched by the user the string MoneyTransfer("Recipient,Amount", 12345, 100) is inscribed, 100 coins will be transferred to the wallet 12345. In such a case the user who signs an external contract will remain not in the know of the transaction. This situation may be excluded if the MoneyTransfer contract requires the additional user's signature upon its calling in of contracts. To do this:

1. Adding a field with the name **Signature** with the *optional* and *hidden* parameters in the *data* section of the *MoneyTransfer* contract, which allow not to require the additional signature in the direct calling of the contract, since there will be the signature in the **Signature** field so far.

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }


2. Adding in the *Signatures* table the entry containing:
•	MoneyTransfer contract name,
•	field names whose values will be displayed to the user, and their text description,
•	text to be displayed upon confirmation.

In the current example it will be enough specifying two fields **Receipient** and **Amount**:


* **Title**: Are you agree to send money this recipient?
* **Parameter**: Receipient Text: Wallet ID
* **Parameter**: Amount Text: Amount (qEGS)

Now, if inserting the *MoneyTransfer(“Recipient, Amount”, 12345, 100)* contract calling in, the system error *“Signature is not defined”* will be displayed. If the contract is called in as follow: *MoneyTransfer(“Recipient, Amount, Signature”, 12345, 100, ”xxx...xxxxx”), the system error will occur upon signature verification. Upon the contract calling in, the following information is verified: ""time of the initial transaction, user ID, the value of the fields specified in the signatures table"", and it is impossible to forge the signature.

In order for the user to see the money transfer confirmation upon the *MoneyTransfer* contract calling in, it is necessary to add a field with an arbitrary name and the type *string*, and with the optional parameter *signature:contractname*. Upon calling in of the enclosed *MoneyTransfer* contract, you just need to forward this parameter. It should also be borne in mind that the parameters for the secured contract calling in must also be described in the *data* section of the external contract (they may be hidden, but they will still be displayed upon confirmation). For instance,

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount    money
          Signature string "signature:send_money"
      }
      func action {
          MoneyTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

When sending a *MyTest* contract, the additional confirmation of the money transfer to the indicated wallet will be requested from user. If other values, such as *MoneyTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)*, are listed in the enclosed contract, the invalid signature error will occur.
