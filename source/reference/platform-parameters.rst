Platform parameters
###################


About platform parameters
=========================


Platform parameters are configuration parameters for the blockchain platform. These parameters apply to the blockchain network and to all ecosystems in the network.

Where platform parameters are stored
------------------------------------

Platform parameters are stored in the ``system_parameters`` table. 

This table is available in the first (default) ecosystem that is created on the blockchain network.


Changing platform parameters
----------------------------
 
Changes to platform parameters must be introduced as a result of voting.

.. todo::

    Link to a topic where this is explained further.



Platform parameters by purpose
==============================


Blockchain network
------------------

Nodes: 

- :ref:`full_nodes`
- :ref:`number_of_nodes`

Node ban: 

- :ref:`incorrect_blocks_per_day`
- :ref:`node_ban_time`
- :ref:`local_node_ban_time`


User interfaces
---------------

Default pages and menus: 

- :ref:`default_ecosystem_page`

.. todo::

    Following params have no description

- :ref:`default_ecosystem_menu`

.. todo::

    Not sure what it does and if it belongs here

- :ref:`default_ecosystem_contract`


Database
--------

Table limits: 

- :ref:`max_columns`
- :ref:`max_indexes`

Block generation
----------------

Time limits: 

- :ref:`gap_between_blocks`
- :ref:`max_block_generation_time`

Transaction limits: 

- :ref:`max_tx_count`
- :ref:`max_block_user_tx`
- :ref:`max_tx_size`

Size limits: 

- :ref:`max_block_size`
- :ref:`max_forsign_size`

Rewards and commissions: 

- :ref:`block_reward`
- :ref:`commission_wallet`
- :ref:`commission_size`

Rollback: 

- :ref:`rb_blocks_1`

Blockchain downloads: 

- :ref:`blockchain_url`

.. todo::

    Following params have no description

- :ref:`new_version_url`


Fuel
----

Fuel parameters: 

- :ref:`fuel_rate`
- :ref:`max_fuel_block`
- :ref:`max_fuel_tx`
- :ref:`size_fuel`

Prices for elements: 

- :ref:`ecosystem_price`

.. todo::

    Following params have no description

- :ref:`column_price`
- :ref:`contract_price`
- :ref:`menu_price`
- :ref:`page_price`
- :ref:`table_price`

Operation costs: 

- :ref:`extend_cost_activate`
- :ref:`extend_cost_address_to_id`
- :ref:`extend_cost_column_condition`
- :ref:`extend_cost_compile_contract`
- :ref:`extend_cost_contains`
- :ref:`extend_cost_contracts_list`
- :ref:`extend_cost_contract_by_name`
- :ref:`extend_cost_contract_by_id`
- :ref:`extend_cost_create_column`
- :ref:`extend_cost_create_ecosystem`
- :ref:`extend_cost_create_table`
- :ref:`extend_cost_deactivate`
- :ref:`extend_cost_ecosys_param`
- :ref:`extend_cost_eval`
- :ref:`extend_cost_eval_condition`
- :ref:`extend_cost_flush_contract`
- :ref:`extend_cost_has_prefix`
- :ref:`extend_cost_id_to_address`
- :ref:`extend_cost_is_object`
- :ref:`extend_cost_join`
- :ref:`extend_cost_json_to_map`
- :ref:`extend_cost_len`
- :ref:`extend_cost_new_state`
- :ref:`extend_cost_perm_column`
- :ref:`extend_cost_perm_table`
- :ref:`extend_cost_pub_to_id`
- :ref:`extend_cost_replace`
- :ref:`extend_cost_sha256`
- :ref:`extend_cost_size`
- :ref:`extend_cost_substr`
- :ref:`extend_cost_sys_fuel`
- :ref:`extend_cost_sys_param_int`
- :ref:`extend_cost_sys_param_string`
- :ref:`extend_cost_table_conditions`
- :ref:`extend_cost_update_lang`
- :ref:`extend_cost_validate_condition`


Platform parameters
===================

.. _block_reward:

block_reward
------------

    Value of reward that is awarded on block generation to the node that generated the block.


    .. todo::

        Where this is paid from? Measurement units?
        Where to this is awarded?


.. _blockchain_url:

blockchain_url
--------------
    
    URL of the full blockchain file. 

    This URL can be used to download the blockchain instead of receiving it from nodes.


.. _column_price:

column_price
------------

    Fuel cost for creating a new table column.

.. _commission_size:


commission_size
---------------
    
    Commission size, in percents.

    This commission is collected for all operations and transferred to the account specified in the :ref:`commission_wallet` parameter.


    .. todo::

        Check that it really works like so. What operations have commission?


.. _commission_wallet:

commission_wallet
-----------------

    Account that collects commission for operations.

    Size of the commission is specified in the :ref:`commission_size` parameter.


.. _contract_price:

contract_price
--------------

    Fuel cost for creating a new contract.


.. _default_ecosystem_contract:

default_ecosystem_contract
--------------------------

    Source code of the default ecosystem contract.

    .. todo::

        What this contract does?


.. _default_ecosystem_menu:

default_ecosystem_menu
----------------------

    Source code of the default ecosystem menu.


.. _default_ecosystem_page:

default_ecosystem_page
----------------------

    Source code of the default ecosystem page.

    .. todo::

        Default page for all new ecosystems OR page for default ecosystem?

        Same for  default_ecosystem_menu and default_ecosystem_contract.



.. _ecosystem_price:

ecosystem_price
---------------

    Fuel cost for creating a new ecosystem.


.. _extend_cost_activate:

extend_cost_activate
--------------------
    
    Fuel cost of :func:`Activate` function call.


.. _extend_cost_address_to_id:

extend_cost_address_to_id
-------------------------
    
    Fuel cost of :func:`AddressToId` function call.


.. _extend_cost_column_condition:

extend_cost_column_condition
----------------------------
    
    Fuel cost of :func:`ColumnCondition` function call.


.. _extend_cost_compile_contract:

extend_cost_compile_contract
----------------------------
    
    Fuel cost of :func:`CompileContract` function call.


.. _extend_cost_contains:

extend_cost_contains
--------------------
    
    Fuel cost of :func:`Contains` function call.


.. _extend_cost_contracts_list:

extend_cost_contracts_list
--------------------------
    
    Fuel cost of :func:`ContractsList` function call.


.. _extend_cost_contract_by_name:

extend_cost_contract_by_name
----------------------------
    
    Fuel cost of :func:`GetContractByName` function call.

    .. todo::

        In source, not in the product


.. _extend_cost_contract_by_id:

extend_cost_contract_by_id
--------------------------

    Fuel cost of :func:`GetContractById` function call.
    
    .. todo::

        In source, not in the product


.. _extend_cost_create_column:

extend_cost_create_column
-------------------------
    
    Fuel cost of :func:`CreateColumn` function call.


.. _extend_cost_create_ecosystem:

extend_cost_create_ecosystem
----------------------------
    
    Fuel cost of :func:`CreateEcosystem` function call.


.. _extend_cost_create_table:

extend_cost_create_table
------------------------
    
    Fuel cost of :func:`CreateTable` function call.

    .. todo::

        How it works with table_price?


.. _extend_cost_deactivate:

extend_cost_deactivate
----------------------
    
    Fuel cost of :func:`Deactivate` function call.


.. _extend_cost_ecosys_param:

extend_cost_ecosys_param
------------------------
    
    Fuel cost of :func:`EcosysParam` function call.


.. _extend_cost_eval:

extend_cost_eval
----------------
    
    Fuel cost of :func:`Eval` function call.


.. _extend_cost_eval_condition:

extend_cost_eval_condition
--------------------------
    
    Fuel cost of :func:`EvalCondition` function call.


.. _extend_cost_flush_contract:

extend_cost_flush_contract
--------------------------
    
    Fuel cost of :func:`FlushContract` function call.


.. _extend_cost_has_prefix:

extend_cost_has_prefix
----------------------
    
    Fuel cost of :func:`HasPrefix` function call.


.. _extend_cost_id_to_address:

extend_cost_id_to_address
-------------------------
    
    Fuel cost of :func:`IdToAddress` function call.


.. _extend_cost_is_object:

extend_cost_is_object
---------------------
    
    Fuel cost of :func:`IsObject` function call.


.. _extend_cost_join:

extend_cost_join
----------------
    
    Fuel cost of :func:`Join` function call.


.. _extend_cost_json_to_map:

extend_cost_json_to_map
-----------------------
    
    Fuel cost of :func:`JSONToMap` function call.


.. _extend_cost_len:

extend_cost_len
---------------
    
    Fuel cost of :func:`Len` function call.


.. _extend_cost_new_state:

extend_cost_new_state
---------------------
    
    This parameter is deprecated. 

    Use :ref:`extend_cost_create_ecosystem` instead.


.. _extend_cost_perm_column:

extend_cost_perm_column
-----------------------
    
    Fuel cost of :func:`PermColumn` function call.


.. _extend_cost_perm_table:

extend_cost_perm_table
----------------------
    
    Fuel cost of :func:`PermTable` function call.


.. _extend_cost_pub_to_id:

extend_cost_pub_to_id
---------------------
    
    Fuel cost of :func:`PubToID` function call.


.. _extend_cost_replace:

extend_cost_replace
-------------------
    
    Fuel cost of :func:`Replace` function call.


.. _extend_cost_sha256:

extend_cost_sha256
------------------
    
    Fuel cost of :func:`Sha256` function call.
    

.. _extend_cost_size:

extend_cost_size
----------------
    
    Fuel cost of :func:`Size` function call.
    

.. _extend_cost_substr:

extend_cost_substr
------------------
    
    Fuel cost of :func:`Substr` function call.


.. _extend_cost_sys_fuel:

extend_cost_sys_fuel
--------------------
    
    Fuel cost of :func:`SysFuel` function call.


.. _extend_cost_sys_param_int:

extend_cost_sys_param_int
-------------------------
    
    Fuel cost of :func:`SysParamInt` function call.


.. _extend_cost_sys_param_string:

extend_cost_sys_param_string
----------------------------
    
    Fuel cost of :func:`SysParamString` function call.
    

.. _extend_cost_table_conditions:

extend_cost_table_conditions
----------------------------
    
    Fuel cost of :func:`TableConditions` function call.
    

.. _extend_cost_update_lang:

extend_cost_update_lang
-----------------------
    
    Fuel cost of :func:`UpdateLang` function call.


.. _extend_cost_validate_condition:

extend_cost_validate_condition
------------------------------
    
    Fuel cost of :func:`ValidateCondition` function call.


.. _fuel_rate:

fuel_rate
---------

    Exchange rate for fuel.


    .. todo::

        Check if true:

        token_cost = fuel_cost \* fuel_rate


.. _full_nodes:

full_nodes
----------
    
    List of validating nodes of the blockchain network.

    Format:

        ``[["host1:port","-1222","nodepub1"], ["host2:ip","-1222","nodepub2"]]``

        ``host1:port`` is the address of the host where transactions and new blocks are sent; this address can also be used to receive the full blockchain starting from the first block.

        ``-1222`` is the account identifier that receives the commission; if this account oes not exist, the commission is not collected.

        ``nodepub1`` public key of the node; this key is required to check block signatures.


.. _gap_between_blocks:

gap_between_blocks
------------------

    Amount of time, in seconds, allotted to the validating node to create a new block.

    Minimum value for this parameter is ``1`` (1  second).

.. _incorrect_blocks_per_day:

incorrect_blocks_per_day
------------------------

    Amount of incorrect blocks per day that a node may generate before it is banned.

    .. todo::

        How global ban works?


.. _local_node_ban_time:

local_node_ban_time
-------------------

    Local ban period for nodes, in ms.
    
    .. todo::

        How local ban works?


.. _max_block_generation_time:

max_block_generation_time
-------------------------

    Maximum amount of time, in ms, allowed for block generation.


.. _max_block_size:

max_block_size
--------------

    Maximum block size.

    .. todo::

        Size in bytes?


.. _max_block_user_tx:

max_block_user_tx
-----------------

    Maximum number of user transactions in one block.

    .. todo::

        What's non-user transactions?


.. _max_columns:

max_columns
-----------

    Maximum number of columns in tables.

    .. todo::

        preset id column included?


.. _max_forsign_size:

max_forsign_size
----------------

    TBD

    .. todo::

        What is forsign? Some part of transaction for signature?

        // MaxForsignSize is the maximum size of the forsign of transaction


.. _max_fuel_block:

max_fuel_block
--------------

    Maximum amount of fuel that can be used in a single block.

    .. todo::

        How it works?

        // MaxBlockFuel is the maximum fuel of the block


.. _max_fuel_tx:

max_fuel_tx
-----------

    Maximum amonut of fuel that can be used in a single transaction.

    .. todo::

        How it works?

        // MaxTxFuel is the maximum fuel of the transaction


.. _max_indexes:

max_indexes
-----------

    Maximum number of index fields in a table.


.. _max_tx_count:

max_tx_count
------------

    Maimum number of transactions in a single block.

    .. todo::

        Check that this is true.


.. _max_tx_size:

max_tx_size
-----------

    Maximum transaction size.

    .. todo::

        Measurement units?


.. _menu_price:

menu_price
----------

    Fuel cost for creating a new menu.


.. _new_version_url:

new_version_url
---------------

    TBD

    .. todo::

        Version of what?


.. _node_ban_time:

node_ban_time
-------------

    Global ban period for nodes, in ms.
    
    .. todo::

        How global ban works?


.. _number_of_nodes:

number_of_nodes
---------------

    Maximum number of validating nodes in the :ref:`full_nodes` parameter.

.. _page_price:


page_price
----------

    Fuel cost for creating a new page.

.. _rb_blocks_1:


rb_blocks_1
-----------

    TBD

    .. todo::

        What this parameter does?
        
        // RbBlocks1 rollback from queue_bocks


.. _size_fuel:

size_fuel
---------

    Fuel cost of 1024 bytes of the transaction data.

    .. todo::

        found in source, not in product


.. _table_price:

table_price
-----------

    Fuel cost for creating a new table.