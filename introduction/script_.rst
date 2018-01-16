################################################################################
Smart-contracts
################################################################################
.. contents::
  :local:
  :depth: 2
  
********************************************************************************
Structure of the contract
********************************************************************************


Data section
==============================
The contract input data, as well as the parameters of the form for the reception of the data are described in the **data** section. 
The data are listed line by line: first, the variable name is specified (only variables, but not arrays are transferred), then the type and the parameters for building of the interface form are indicated optionally through a gap in double quotation marks:

*	*hidden* - hidden element of the form,
*	*optional* - form element without obligatory filling in,
*	*date* - field of the date and time selection,
*	*polymap* - map with the selection of coordinates and areas,
*	*map* - map with the ability to mark the place,
*	*image* - images upload,
*	*text* - entry of the text of HTML-code in the textarea field,
*crypt:Field* - create and encrypt a private key for the destination specified in the *Field* field. If only * crypt * is specified, then the private key will be created for the user who signs the contract,
*	*address* - field for input of the wallet address,
*	*signature:contractname* - a line to display the contractname contract, which requires the signatures (it is discussed in detail in a special description section).

.. code:: js

  contract my {
    data {
        Name string 
        RequestId address
        Photo bytes "image optional"
        Amount money
        Private bytes "crypt:RequestId"
    }
    ...
  }
    
Conditions section
==============================
Validation of the data obtained is performed in the section. The following commands are used to warn of the presence of errors: ``error``, ``warning``, ``info``. In fact, all they generate an error that stops the contract operation, but display different messages in the interface: *critical error*, *warning*, and *informative error*. For instance, 

.. code:: js

  if fuel == 0 {
        error "fuel cannot be zero!"
  }
  if money < limit {
        warning Sprintf("You don't have enough money: %v < %v", money, limit)
  }
  if idexist > 0 {
        info "You have been already registered"
  }
Action section
==============================

Variables in the contract
==============================

Enclosed contracts
==============================

Contracts with signature
==============================
Since the language of contracts writing allows performing enclosed contracts, it is possible to fulfill such an enclosed contract without the knowledge of the user who has run the external contract that may lead to the user's signature of transactions unauthorized by it, let's say the transfer of money from its account.

Let's suppose there is a TokenTransfer Contract *TokenTransfer*:

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

If in a contract launched by the user the string ``TokenTransfer("Recipient,Amount", 12345, 100)`` is inscribed, 100 coins will be transferred to the wallet 12345. In such a case the user who signs an external contract will remain not in the know of the transaction. This situation may be excluded if the TokenTransfer contract requires the additional user's signature upon its calling in of contracts. To do this:

1. Adding a field with the name **Signature** with the ``optional`` and ``hidden`` parameters in the *data* section of the *TokenTransfer* contract, which allow not to require the additional signature in the direct calling of the contract, since there will be the signature in the **Signature** field so far.

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }


2. Adding in the *Signatures* table (on the page **Signatures** of Apla client) the entry containing:

•	*TokenTransfer* contract name,
•	field names whose values will be displayed to the user, and their text description,
•	text to be displayed upon confirmation.
  
In the current example it will be enough specifying two fields **Receipient** and **Amount**:

* **Title**: Are you agree to send money this recipient?
* **Parameter**: Receipient Text: Wallet ID
* **Parameter**: Amount Text: Amount (qEGS)

Now, if inserting the *TokenTransfer(“Recipient, Amount”, 12345, 100)* contract calling in, the system error ``“Signature is not defined”`` will be displayed. If the contract is called in as follow: ``TokenTransfer("Recipient, Amount, Signature", 12345, 100, "xxx...xxxxx")``, the system error will occur upon signature verification. Upon the contract calling in, the following information is verified: *time of the initial transaction, user ID, the value of the fields specified in the signatures table*, and it is impossible to forge the signature.

In order for the user to see the money transfer confirmation upon the *TokenTransfer* contract calling in, it is necessary to add a field with an arbitrary name and the type ``string``, and with the optional parameter ``signature:contractname``. Upon calling in of the enclosed *TokenTransfer* contract, you just need to forward this parameter. It should also be borne in mind that the parameters for the secured contract calling in must also be described in the ``data`` section of the external contract (they may be hidden, but they will still be displayed upon confirmation). For instance,

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount  money
          Signature string "signature:TokenTransfer"
      }
      func action {
          TokenTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

When sending a *MyTest* contract, the additional confirmation of the money transfer to the indicated account will be requested from user. If other values, such as ``TokenTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)``, are listed in the enclosed contract, the invalid signature error will occur.

