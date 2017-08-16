################################################################################
REST API
################################################################################

Rough template of API commands for EGAAS.

The command is called when **/api/v1/command/[param]** is accessed, where **command** is the command name, **param** is an optional parameter, for example, the name of the resource being changed or received. The server response is represented in JSON format.

In each response, the **error** parameter is returned, which is empty if the command is successfully executed or contains an error text. 
*Is it required to enter additional response codes other than 200?*
Such as Bad Request (400), Unauthorized (401), Not Found (404). 
Except the error field, all commands that send transactions also return the **hash** field with the hash of that posted transaction. Then, with this hash and the **txstatus** command, you can get the number of the blockchain where the transaction or processing error was sealed.

At this point, at the first request, the server opens a session for that user and returns it to Cookie in gosessionix. Therefore, for current compatibility, you need to pass this cookie value additionally with each request. For example, **Cookie: gosessionid=34673476347834**. Subsequently, you can go to JWT https://jwt.io/

********************************************************************************
Common API commands
********************************************************************************

getuid
==============================
**GET** Returns a unique value , which you need to sign with your private key and send back to the server using the **auth** command.

.. code:: 
    
    GET
    /api/v1/getuid
    
    Possible response
    
    .. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "uid": "28726874268427424",
        "error": ""
    }
    
login
==============================
**POST** User authentication. You first need to call the **getuid** command to get a unique value and sign it. If successful, the corresponding address is returned in the format XXXX-XXXX-...-XXX. If 0 is specified as the ecosystem identifier, then the commands that work within the ecosystem will be unavailable.

.. code:: 
    
    POST
    /api/v1/login
    pubkey: hex public key
    signature: hex signed uid
    state: state identifier
    
    Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "address": "1234-....-3424",
        "error": ""
    } 
    
    signtest
==============================
**POST** the command signs a string with the specified private key. It should be used solely for API testing, because in real work a private key should not be passed to the server. Returns the *signature* - hexadecimal signature and *pubkey* - public key for this private key.

.. code:: 
    
    POST
    /api/v1/signtest
    private: hex private key
    forsign: signature string
    pubkey: hex public key
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "signature": "0011fa...",
        "pubkey": "324bd7..."
    }      



********************************************************************************
API commands for working with EGS
********************************************************************************
    
balance
==============================
**GET** Gets the EGS balance of the specified wallet. The wallet number is indicated by the second parameter and can be represented in any format- int64, uint64, XXXX-...-XXXX. In the *amount* parameter, the amount in the wallet is returned in qEGS, while in the *egs* parameter, the same value is returned in EGS.

.. code:: 
    
    GET
    /api/v1/balance/1234-...-4673
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "amount": "123450000000000000000",
        "egs": "123.45"
        "error": ""
    } 
    
    sendegs 
==============================
**POST** Sends EGS from one wallet to another. Before calling this command, the **prepare/sendegs** request must be sent, and then, after signing return parameter *forsign*, you must send the same request with the signature field in which the *signature* is written.

.. code:: 
    
    POST
    /api/v1/sendegs
    pubkey: hex public key
    recipient - address of the recipient
    amount - amount transferred to qEGS
    commission - comission qEGS
    comment - comment
    signature - hex signature
    time - time returned by prepare

Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634",
        "error": ""
    }      

********************************************************************************
API service commands
********************************************************************************

prepare/{tx}
==============================
**POST/PUT** Sends a request to get a string to sign a transaction, where **{tx}** is the name of the transaction for which you want to return the string for signature. In the forsign parameter, the string to be signed is returned. The time parameter, which needs to be posted together with the signature field, is also returned. The POST or PUT method must match with the method by which the transaction itself will be posted.

.. code:: 
    
    POST
    /api/v1/prepare/sendegs
    pubkey: hex public key
    recipient - recipient’s address
    amount - amount transferred in qEGS
    commission - qEGS commission
    comment - comment

Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "time": 423523768,
        "forsign": "......", 
        "error": ""
    }      

txstatus/{hash}
==============================
**GET** returns the block number or error of the posted transaction with the hash. If the returned *blockid* and *error* values are empty, then the transaction has not yet been sealed in the blockchain.

.. code:: 
    
    GET
    /api/v1/txstatus/2353467abcd7436ef47438
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "blockid": "423523768",
        "error": ""
    }      

********************************************************************************
API commands for content
********************************************************************************

content/{menu|page}/{name}/[?global=1]
==============================
**GET** returns the HTML code of a page or menu named **{name}**, which is returned by the template engine.

.. code:: 
    
    GET
    /api/v1/content/page/default
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "html": "......",
        "error": ""
    }      
    
    
menu/{name}/[?global=1]
==============================
**GET** to get the fields specified in the second parameter **{name}** menu. By default, the menu for the current state is returned. If an additional global parameter is specified, the *global* menu will be returned.

.. code:: 
    
    GET
    /api/v1/menu/government
    
Possible answer

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "government",
        "value": "MenuItem()....", 
        "conditions": "MainCondition()",
        "error": ""
    }      

menu
==============================
**POST** adds a new menu. You first need to call the **prepare/menu** command (POST) and sign the returned forsign field. 

.. code:: 
    
    POST
    /api/v1/menu
    name - menu name
    value - menu code
    conditions - permission for edit
    global - point 1, if you need to add the global menu. Otherwise a menu will be added to the current state.
    signature - hex signature
    time - time returned by prepare
   
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      

menu/{name}/[?global=1]
==============================
**PUT** replaces the existing menu named **{name}**. You first need to call the **prepare/menu** command (PUT) and sign the returned forsign field. If the global menu is being replaced, you should add the global parameter.

.. code:: 
    
    PUT
    /api/v1/menu/government
    value - menu template
    conditions - permission for edit
    signature - hex signature
    time - time returned by prepare
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      

appendmenu/{name}/[?global=1]
==============================
**PUT** adds items to an existing menu named **{name}**. You first need to call the **prepare/appendmenu** command (PUT) and sign the returned forsign field. If the global menu is being replaced, you should add the *global* parameter.

.. code:: 
    
    PUT
    /api/v1/appendmenu/government
    value - the added menu template
    signature - hex signature
    time - time returned by prepare
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }          

menulist/[?limit=...&offset=...&global=1]
==============================
**GET** returns a list of menus. You can specify the offset and the number of menus requested. If you need global menus, you should add the *global* parameter. You can also specify the offset and the number of items received.

.. code:: 
    
    GET
    /api/v1/menulist/
    
Вариант ответа

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "menu_default",
        }, 
        { 
            "name": "government",
       }, 
        ]
    }      


page/{name}/[?global=1]
==============================
**GET** to gets the fields of a page named **{name}**. By default, the page for the current state is returned. If an additional *global* parameter is specified, the global page will be returned. In addition to *name, value* and *conditions*, the name of the menu related to this page is also returned.

.. code:: 
    
    GET
    /api/v1/page/default
    
    Possible response
    
.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "default",
        "value": "Div:...", 
        "menu": "government", 
        "conditions": "MainCondition()",
        "error": ""
    }      

page
==============================
**POST** to add a new page. You first need to call the **prepare/page** (POST) command and sign the returned *forsign* field.

.. code:: 
    
    POST
    /api/v1/page
    name - name of the page
    menu - name of related menu 
    value - page template
    conditions - permission for edit
    global - point 1, if you need to add the global page. Otherwise, a page will be added to the current state.
    signature - hex signature
    time - time returned by prepare
    
Вариант ответа

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      

page/{name}/[?global=1]
==============================
**PUT** replaces an existing page named **{name}**. You first need to call the prepare/page command (PUT) and sign the returned forsign field. If the global page is being replaced, you should add the *global* parameter.


.. code:: 
    
    PUT
    /api/v1/page/default
    value - шаблон страницы
    conditions - права на изменения
    signature - hex подпись
    time - время, возвращенное prepare
    
Вариант ответа

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      

appendpage/{name}/[?global=1]
==============================
**PUT** to add a template to an existing page named **{name}**. You first need to call the **prepare/appendpage** command (PUT) and sign the returned forsign field. If the global page is being replaced, you should add the *global* parameter.

.. code:: 
    
    PUT
    /api/v1/appendpage/government
    value - the added page template
    signature - hex signature
    time - time returned by prepare
    
    Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }          

pagelist/[?limit=...&offset=...&global=1]
==============================
**GET** returns a list of pages. You can specify the offset and number of pages requested. If you need global pages, you should add the *global* parameter. You can also specify the offset and the number of items received.

.. code:: 
    
    GET
    /api/v1/pagelist/
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "MyPage",
            "menu": "government",
        }, 
        { 
            "name": "government",
            "menu": "government",
       }, 
        ]
    }      

lang/{name}
==============================
**GET** to get translations for a language resource named **{name}**. Resources for the current state are returned.

.. code:: 
    
    GET
    /api/v1/lang/confirm
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "confirm",
        "trans": "{\"en\": \"Confirm\", \"nl\": \"Bevestig\", \"ru\": \"Подтвердить\"}"
    }      

lang
==============================
**POST** to adds a new language resource. You first need to call the **prepare/lang** command (POST) and sign the returned forsign field.

.. code:: 
    
    POST
    /api/v1/lang
    name - name of the language resource
    trans - language resources in JSON format. For example, {"en": "Confirm", "nl": "Bevestig", "fr": "Confirmer"}
    signature - hex signature
    time - time returned by prepare
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      

lang/{name}
==============================
**PUT** to replaces an existing language resource named **{name}**. You first need to call the **prepare/lang** command (PUT) and sign the returned forsign field.

.. code:: 
    
    PUT
    /api/v1/lang/confirm
    trans - language resources in JSON format. For example, {"en": "Confirm", "nl": "Bevestig", "fr": "Confirmer"}
    signature - hex signature
    time - time returned by prepare
    
Possible answer

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
        "error": ""
    }      
    
langlist/[?limit=...&offset=...]
==============================
**GET** returns a list of language resources. You can specify the offset and the number of resources requested.

.. code:: 
    
    GET
    /api/v1/langlist/?limit=2
    
Possible response

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "Yes",
            "trans": {"ru": "Да", "en": "Yes"}
        }, 
        { 
            "name": "No",
            "trans": {"ru": "Нет", "en": "No"}
        }, 
        ]
    }      

********************************************************************************
Команды API для работы с контрактами
********************************************************************************
