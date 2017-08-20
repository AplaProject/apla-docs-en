################################################################################
Apps
################################################################################
The app for the eGaaS platform is a standalone software solution that performs a specific action or a set of actions within a certain activity. The app consists of

* Contracts that implement its functionality,
*	Database tables
* Page and menu for data input and display

The applications have an open structure; therefore they can be expanded by adding new contracts, pages and tables.

Applications are developed in the eGaaS software client, which ensures:

*	Opening of new tables and adding of the required number of columns (menu item Tables) to them. When creating table columns in the applications, the following formats are supported:

      * Text – text (without index support);
      * Numbers – integer values bigint (8 bytes);
      *	Date/Time – date and time in YYYY-MM-DD HH:MM:SS format;
      * Varchar(32) – a string of no more than 32 characters;
      *	Money – integer (numeric);
      *	Double – a double-precision floating-point number.
*	Creating new contracts (menu item Smart contracts);
*	Creating new page and menu templates (menu item Interface);
*	Adding and editing parameters of the configuration table (link State parameters on the Tables page);
*	Adding and editing edit permissions for tables, table columns, contracts, pages and menu, parameters of the setup table.

Special languages of the eGaaS platform are used for writing contracts and pages of templates.

********************************************************************************
Example of an application
********************************************************************************
We will consider an example, where a simple test application is created and which facilitates the following action:

*	Opening of personal accounts by the Central Bank;
*	Depositing of money in an account by the Central Bank;
*	Transfer of money between accounts.

First of all, we create a table called accounts with the following columns:

*	**id** – account number *Numbers + Index*;
*	**amount** – amount of money in the account, *Money + Index*;
*	**citizen_id** – the identification number of the citizen *Numbers + Index*.

Next, we create contracts needed to implement each of the actions.

**Opening a new account**

.. code:: js

	contract AddCitizenAccount {
	data {
		CitizenId string
	}
	func conditions {
	    
	    	$citizen_id = AddressToId($CitizenId)
		if $citizen_id == 0 {
			warning "not valid citizen id"
		}
		
		CentralBankConditions()
	
	}
	func action {
		
		DBInsert(Table("accounts"), "citizen_id", $citizen_id)
		}
	} 
  
  
In the conditions section, the citizen's id is verified and the user calling this contract is checked to see whether he/she has permission to open an account. To this end, a special contract CentralBankConditions is created.

**Authorization to sign contracts on behalf of the Central Bank**

.. code:: js

	contract CentralBankConditions {
	data {	}
	func conditions	{
	   if StateValue("gov_account") != $citizen
	   {
	       	warning "You have no right to this action"
	   }
	}
	func action {	}
	}
  
Now, in this contract, the “creator of the state” – a user with the id specified in the *gov_account* row of configuration table **State parameters** – is authorized to perform actions on behalf of the Central Bank. From henceforth, by changing this contract, signature rights can be transferred to a citizen holding the relevant position in the bank. This contract in the app plays the role of a smart law, which can be amended by a representative body.

**Depositing money in an account**

.. code:: js

	contract RechargeAccount {
	data {
		AccountId int
		Amount money
		}
	
	func conditions	{
		CentralBankConditions()
		}

	func action {
		var recipient_amount money
            	recipient_amount = DBAmount(Table("accounts"), "id", $AccountId)
            	recipient_amount = recipient_amount + $Amount
            	DBUpdate(Table("accounts"), $AccountId, "amount", recipient_amount)
		}
	}
  
The citizen's account number and the amount of money being deposited are indicated in the contract as input data. In the *conditions* section, the user calling this contract is checked whether  he/she is authorized to act on behalf of the Central Bank. In the *action* section, the procedure for depositing money in the account is implemented.

**System contract for money transfer from account to account**

A separate system contract for money transfer is necessary first of all to prevent unauthorized access to accounts. It is this system contract that is indicated in the list of contracts authorized to edit the value of the *amount* column in the **accounts** table. To do this, when editing a table, the function *ContractAccess(“MoneyTransfer”,”RechargeAccount”)* should be entered in the *Permissions* field in the *amount* parameter. After that, only these two contracts will be able to change accounts, and transactions between accounts in all applications will have to be implemented only by calling the **MoneyTransfer contract**.

A system contract is also required in order to prevent unauthorized debiting of money from a user's account via hidden nested contracts. To this end, the money transfer system contract uses a signature verification mechanism described in the Contracts with signature section :ref:`id8`

.. code:: js

	contract MoneyTransfer {
	data {
		Amount money
		SenderAccountId int
		RecipientAccountId int
		Signature string "optional hidden"
		}
	func conditions {
    
	    	 if DBAmount(Table("accounts"), "id", $SenderAccountId) < $Amount {
			warning "Not enough money"
	    	 }
		}
	func action {

		    var sender_amount money
		    sender_amount = DBAmount(Table("accounts"), "id", $SenderAccountId)
		    sender_amount = sender_amount - $Amount
		    DBUpdate(Table("accounts"), $SenderAccountId, "amount",  sender_amount)

		    var recipient_amount money
		    recipient_amount = DBAmount(Table("accounts"), "id", $RecipientAccountId)
		    recipient_amount = recipient_amount + $Amount
		    DBUpdate(Table("accounts"), $RecipientAccountId, "amount", recipient_amount)

		}
	}
  
Inserted in the contract is the line *Signature string “optional hidden”*, which calls the transaction confirmation window (for more details, see “Contracts with signature”). In the *conditions* section, the account is checked to see whether there is enough money in it.

**User contract for money transfer from account to account**

This is the main contract of the app, which implements money transfer, calling system contract **MoneyTransfer**.

.. code:: js

	contract SendMoney {
	data {
		RecipientAccountId int 
		Amount money
		Signature string "signature:MoneyTransfer"
		}
	func conditions {
	 	$sender_id = DBIntExt(Table("accounts"), "id", $citizen, "citizen_id")
	    	if $sender_id==$RecipientAccountId
	    	{
	        	warning("You can not send money to your own account")
	    	}    
		}
	func action {
		MoneyTransfer("SenderAccountId,RecipientAccountId,Amount,Signature",$sender_id,$RecipientAccountId,$Amount,$Signature)
		}
	}

For the contracts described (except **MoneyTransfer** and **CentralBankConditions**, which are used as nested ones), interface forms will have to be created for data input and contract call.

First of all, we create a new Central Bank page: go to *Interface* in the menu option of program agent eGaaS, then click the *addPage* button. In the appropriate fields, enter the name of the **CentralBank page**, the necessary navigation elements and two panels to be used to call a contract:

.. code:: js

	Title : Central bank
	Navigation( LiTemplate(government, Government),Central bank)
	MarkDown: ## Accounts 

	Divs(md-4, panel panel-default panel-body data-sweet-alert)
	    Form()
		Legend(" ", "Add citizen account")

		Divs(form-group)
		    Label("Citizen ID")
		    InputAddress(CitizenId, "form-control input-lg m-b")
		DivsEnd:

		TxButton{ Contract: AddCitizenAccount, Name: Add, OnSuccess: "template, CentralBank" }
	    FormEnd:
	DivsEnd:

	Divs(md-4, panel panel-default panel-body data-sweet-alert)
	    Form()
		Legend(" ", "Recharge Account")

		Divs(form-group)
		    Label("Account ID")
		    Select(AccountId, #state_id#_accounts.id, "form-control input-lg m-b")
		DivsEnd:

		Divs(form-group)
		    Label("Amount")
		    InputMoney(Amount, "form-control input-lg")
		DivsEnd:

		TxButton{ Contract: RechargeAccount, Name: Change, OnSuccess: "template,CentralBank" }
	    FormEnd:
	DivsEnd:

	PageEnd:
  
  
Here, it is necessary to note that when calling a contract, the **TxButton** function automatically passes the values of form fields to the contract if their identifiers coincide with the names of the input parameters of the contracts (*CitizenId* for the **AddCitizenAccount** contract, and *AccountId* and *Amount* for the **RechargeAccount contract**).

To access the created **CentralBank page**, add an item to one of the existing menus, for example, *government*: go to menu editing (from the *Interface* page or from the **CentralBank** page editor) and add the following string:

.. code:: js

	MenuItem(CentralBank, load_template, CentralBank)

Also, in the **CentralBank** page editor, specify the menu that will be displayed when you open the Central Bank page (drop-down list *Menu*) – in our example, it is the menu *government*.

Now you only need to open the citizen’s page **dashboard_default** for editing. Add to it two panels for displaying the account number and balance and a panel for calling the money transfer contract:

.. code:: js

	Divs(md-6)
	     Divs()
	     WiBalance(GetOne(amount, #state_id#_accounts, "citizen_id", #citizen#), StateValue(currency_name) )
	     DivsEnd:
	     Divs()
	     WiAccount( GetOne(id, #state_id#_accounts, "citizen_id", #citizen#) )
	     DivsEnd:
	  DivsEnd:


	 Divs(md-6, panel panel-default panel-body data-sweet-alert)
	    Form()
		Legend(" ", "Send Money")

		Divs(form-group)
		    Label("Account ID")
		    Select(RecipientAccountId, #state_id#_accounts.id, "form-control  m-b")
		DivsEnd:

		Divs(form-group)
		    Label("Amount")
		    InputMoney(Amount, "form-control")
		DivsEnd:

		TxButton{ Contract: SendMoney, OnSuccess: "template,dashboard_default,global:0" }
	    FormEnd:
	DivsEnd:

Now if you have the permissions prescribed in the **CentralBankConditions** smart law, you can open accounts for citizens in the **Centralbank** page and recharge the accounts with some amounts. After that, the citizens will be able to transfer money from an account to another account.

As already noted in the description of applications, they are open standalone modules. If necessary, contracts and forms for opening corporate accounts and other forms of accounts can be added to the application.
