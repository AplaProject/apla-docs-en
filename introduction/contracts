************************************************
Contracts
************************************************

This describes contracts, which are created by default on product installation. All of these contracts are created in the first ecosystem, that's why you need to specify their full name to call them from other ecosystems, for instance, **@1NewContract**.

NewEcosystem
==============================
This contract creates a new ecosystem. To get an identifier of the newly created ecosystem, take the *result* field, which will return in txstatus. 

Parameters
   
* *Name string "optional"* - name for the ecosystem. This parameter can be set and/or chanted later.

MoneyTransfer
==============================
This contract transfers money from the current wallet in the current ecosystem to a specified wallet.

Parameters

* *Recipient string* - recipient's wallet in any format – a number or XXXX-....-XXXX.
* *Amount    string* - transaction amount in qAPL.
* *Comment   string "optional"* - comments.

NewContract
==============================
This contract creates a new contract in the current ecosystem.

Parameters

* *Value string* - text of the contract or contracts.
* *Conditions string* - contract change conditions.
* *Wallet string "optional"* - identifier of a wallet that can activate the contract later.
* *TokenEcosystem int "optional"* - identifier of the ecosystem, which currency will be used for transactions when the contract is activated.

EditContract
==============================
Editing the contract in the current ecosystem.

Parameters
      
* *Id int* - ID of the contract to be edited.
* *Value string* - text of the contract or contracts.
* *Conditions string* - condition for change.

ActivateContract
==============================
Activation of a contract in the current ecosystem. Contracts can be activated only from the wallet, which was specified when the contract was created. After the contract is activated, this wallet will pay for execution of this contract.

Parameters
      
* *Id int* - ID of the contract to activate.

NewParameter
==============================
This contract adds a new parameter to the current ecosystem.

Parameters

* *Name string* - parameter name.
* *Value string* - parameter value.
* *Conditions string

EditParameter
==============================
This contract changes an existing parameter in the current ecosystem.

Parameters

* *Name string* - name of the parameter to be changed.
* *Value string* - new value.
* *Conditions string* - new condition for parameter change.

NewMenu
==============================
This contract adds a new menu in the current ecosystem.

Parameters

* *Name string* - menu name.
* *Value string* - menu text.
* *Title string "optional"* - menu header.
* *Conditions string* - condition for menu change.

EditMenu
==============================
This contract changes an existing menu in the current ecosystem.

Parameters

* *Id int* - ID of the menu to be changed.
* *Value string* - new value.
* *Title string "optional"* - menu header.
* *Conditions string* - new condition for menu change.

AppendMenu
==============================
This contract adds text to an existing menu in the current ecosystem.

Parameters

* *Id int* - ID of the menu to be changed.
* *Value string* - string to be added to the menu.

NewPage
==============================
This contract adds a new page in the current ecosystem.

Parameters

* *Name string* - page name.
* *Value string* - page text.
* *Menu string* - name of the menu, attached to this page.
* *Conditions string* - condition for change.

EditPage
==============================
This contract changes an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed.
* *Value string* - new page value.
* *Menu string* - name of the new menu on the page.
* *Conditions string* - new condition for change.

AppendPage
==============================
The contract adds text to an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed.
* *Value string* - value that needs to be added to the page.

NewBlock
==============================
This contract adds a new block with a template to the current ecosystem. These blocks can be used in the template engine using the Input function.

Parameters

* *Name string* - block name.
* *Value string* - block text.
* *Conditions string* - condition for change.

EditBlock
==============================
This contract changes an existing block in the current ecosystem.

Parameters

* *Id int* - ID of the block to be changed.
* *Value string* - new block value.
* *Conditions string* - new condition for change.

NewTable
==============================
This contract adds a new table in the current ecosystem. 
Parameters

* *Name string* - table name in Latin script. 
* *Columns string* - array of columns in JSON format *[{"name":"...", "type":"...","index": "0", "conditions":"..."},...]*.

  * *name* - column name in Latin script.
  * *type* - type **varchar,bytea,number,datetime,money,text,double,character**.
  * *index* - non-indexed field - "0"; create index - "1".
  * *conditions* - condition for column change.

* *Permissions string* - access conditions in JSON format *{"insert": "...", "new_column": "...", "update": "..."}*.

  * *insert* - condition to insert records.
  * *new_column* - condition to add columns.
  * *update* - condition to change data.


EditTable
==============================
This contract changes access permissions to tables in the current ecosystem. 
Parameters 

* *Name string* - table name in Latin script. 
* *Permissions string* - access permissions in JSON format *{"insert": "...", "new_column": "...", "update": "..."}*.

  * *insert* - condition to insert records.
  * *new_column* - condition to add columns.
  * *update* - condition to change data.   

NewColumn
==============================
This contract adds a new column to a table in the current ecosystem. 

Parameters

* *TableName string* - table name in Latin script. 
* *Name* - column name in Latin script.
* *Type* - type **varchar,bytea,number,money,datetime,text,double,character**.
* *Index* - non-indexed field - "0"; create index - "1".
* *Permissions* - permissions to edit the column.

EditColumn
==============================
This contract changes the rights to change a table column in the current ecosystem. 

Parameters

* *TableName string* - table name in Latin script. 
* *Name* - column name in Latin script.
* *Permissions* - permissions to edit the column.

NewLang
==============================
This contract adds language resources in the current ecosystem. Permissions to add resources are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource in Latin script. 
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example: **{"en": "English text", "ru": "Английский текст"}**.

EditLang
==============================
This contract updates the language resource in the current ecosystem. Permissions to make changes are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource in Latin script. 
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. Например: **{"en": "English text", "ru": "Английский текст"}**.
 
