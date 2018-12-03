.. -- Conditionals Genesis / Apla -------------------------------------------------

.. token naming
.. .. |tokens| replace:: tokens
.. |tokens| replace:: APL tokens


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
- :ref:`node_ban_time_local`


New ecosystems
--------------

Default pages and menus:

- :ref:`default_ecosystem_page`
- :ref:`default_ecosystem_menu`

Default contract:

- :ref:`default_ecosystem_contract`


Database
--------

Table limits:

- :ref:`max_columns`
- :ref:`max_indexes`


.. _parameters-block-limits:

Block generation
----------------

Time limits:

- :ref:`gap_between_blocks`
- :ref:`max_block_generation_time`

Transaction number limits:

- :ref:`max_tx_block`
- :ref:`max_tx_block_per_user`

Size limits:

- :ref:`max_tx_size`
- :ref:`max_block_size`
- :ref:`max_forsign_size`

Fuel limits:

- :ref:`max_fuel_block`
- :ref:`max_fuel_tx`

Block rollback:

- :ref:`rollback_blocks`


Fuel and currencies
-------------------

Rewards and commission:

- :ref:`block_reward`
- :ref:`commission_wallet`
- :ref:`commission_size`

Fuel units exchange:

- :ref:`fuel_rate`

Prices for data:

- :ref:`price_tx_data`

Prices for new elements:

- :ref:`price_create_contract`
- :ref:`price_create_menu`
- :ref:`price_create_page`

Prices for operations:

.. hlist::
    :columns: 2

    - :ref:`price_exec_bind_wallet`
    - :ref:`price_exec_address_to_id`
    - :ref:`price_exec_column_condition`
    - :ref:`price_exec_compile_contract`
    - :ref:`price_exec_contains`
    - :ref:`price_exec_contracts_list`
    - :ref:`price_exec_contract_by_name`
    - :ref:`price_exec_contract_by_id`
    - :ref:`price_exec_create_column`
    - :ref:`price_exec_create_ecosystem`
    - :ref:`price_exec_create_table`
    - :ref:`price_exec_unbind_wallet`
    - :ref:`price_exec_ecosys_param`
    - :ref:`price_exec_eval`
    - :ref:`price_exec_eval_condition`
    - :ref:`price_exec_flush_contract`
    - :ref:`price_exec_has_prefix`
    - :ref:`price_exec_id_to_address`
    - :ref:`price_exec_is_object`
    - :ref:`price_exec_join`
    - :ref:`price_exec_json_to_map`
    - :ref:`price_exec_len`
    - :ref:`price_exec_perm_column`
    - :ref:`price_exec_perm_table`
    - :ref:`price_exec_pub_to_id`
    - :ref:`price_exec_replace`
    - :ref:`price_exec_sha256`
    - :ref:`price_exec_size`
    - :ref:`price_exec_substr`
    - :ref:`price_exec_sys_fuel`
    - :ref:`price_exec_sys_param_int`
    - :ref:`price_exec_sys_param_string`
    - :ref:`price_exec_table_conditions`
    - :ref:`price_exec_update_lang`
    - :ref:`price_exec_validate_condition`

Deprecated
----------

Deprecated parameters:

- :ref:`blockchain_url`

Platform parameters
===================


.. _block_reward:

block_reward
------------

    Amount of |tokens| that is awarded to the node that generated a block.

    An account that receives the reward is specified in the :ref:`full_nodes` parameter.

    This parameter is measured in |tokens|.


.. _blockchain_url:

blockchain_url
--------------

    This parameter is deprecated.


.. _commission_size:

commission_size
---------------

    Commission percent.

    This amount of commission is collected from the total contract cost. Commission is applied to the total contract cost in |tokens|.

    Tokens are transferred to the account specified in the :ref:`commission_wallet` parameter.


.. _commission_wallet:

commission_wallet
-----------------

    Account that collects commission for operations.

    Size of the commission is specified in the :ref:`commission_size` parameter.


.. _default_ecosystem_contract:

default_ecosystem_contract
--------------------------

    Source code of the default contract for a new ecosystem.

    This contract provides access rights to the ecosystem founder.


.. _default_ecosystem_menu:

default_ecosystem_menu
----------------------

    Source code of the default menu for a new ecosystem.


.. _default_ecosystem_page:

default_ecosystem_page
----------------------

    Source code of the default page for a new ecosystem.


.. _fuel_rate:

fuel_rate
---------

    Exchange rate for tokens of different ecosystems to fuel units.

    Format for this parameter is:

        ``[["ecosystem_id", "token_to_fuel_rate"], ["ecosystem_id2", "token_to_fuel_rate2"], ...]``

        - ``ecosystem_id``

            Ecosystem identifier.

        - ``token_to_fuel_rate``

            Exchange rate of tokens to fuel units.

    Example:

        ``[["1","1000000000000000"], ["2", "1000"]]``

        One token from ecosystem 1 is exchanged to 1000000000000000 fuel units. One token from ecosystem 2 is exchanged to 1000 fuel units.


.. _full_nodes:

full_nodes
----------

    List of validating nodes of the blockchain network.

    Format for this parameter is:

        ``[["host:port","wallet_id","node_pub"], ["host2:port2","wallet_id2","node_pub2"]]``

        - ``host:port``

            Address and port of the node host.

            Transactions and new blocks are sent to this host. This address can also be used to obtain the full blockchain starting from the first block.

        - ``wallet_id``

            Wallet (account identifier) that receives rewards for generating new blocks and processing transactions.

        - ``node_pub``

            Public key of the node. This key is used to check block signatures.


.. _gap_between_blocks:

gap_between_blocks
------------------

    Amount of time, in seconds, that a node can use to create a new block.

    This parameter is a network parameter. All nodes in the network use it to determine when to generate new blocks. If a node did not create a block in this time period, the turn passes to the next node in a list of validating nodes.

    Minimum value for this parameter is ``1`` (one second).

    .. todo::

        How it works with max_block_generation_time?


.. _incorrect_blocks_per_day:

incorrect_blocks_per_day
------------------------

    Amount of incorrect blocks per day that a node may generate before it is banned from the network.

    When more than half of nodes in a network have received this amount of incorrect blocks from a certain  node, this node is banned from the network for :ref:`node_ban_time` amount of time.


.. _max_block_generation_time:

max_block_generation_time
-------------------------

    Maximum amount of time that a node may spend to generate a block, in ms.

    .. todo::

        How it works with gap_between_blocks?


.. _max_block_size:

max_block_size
--------------

    Maximum block size, in bytes.


.. _max_columns:

max_columns
-----------

    Maximum number of columns in tables.

    The predefined ``id`` column is not included in this maximum.


.. _max_forsign_size:

max_forsign_size
----------------

    Maximum size, in bytes, of a forsign (string to be signed) generated for a transaction.

    .. todo::

        Better explain what a forsign is.


.. _max_fuel_block:

max_fuel_block
--------------

    Maximum total fuel cost of a single block.


.. _max_fuel_tx:

max_fuel_tx
-----------

    Maximum total fuel cost of a single transaction.


.. _max_indexes:

max_indexes
-----------

    Maximum number of index fields in a table.


.. _max_tx_block:

max_tx_block
------------

    Maimum number of transactions in a single block.


.. _max_tx_block_per_user:

max_tx_block_per_user
---------------------

    Maximum number of transactions in one block that belong to one account.


.. _max_tx_size:

max_tx_size
-----------

    Maximum transaction size, in bytes.


.. _node_ban_time:

node_ban_time
-------------

    Global ban period for nodes, in ms.

    When more than half of nodes in a network have received :ref:`incorrect_blocks_per_day` amount of blocks from a certain node, this node is banned from the network for the specified amount of time.


.. _node_ban_time_local:

node_ban_time_local
-------------------

    Local ban period for nodes, in ms.

    When a node receives an incorrect block from another node, it bans the sender node locally for this amount of time.


.. _number_of_nodes:

number_of_nodes
---------------

    Maximum number of validating nodes in the :ref:`full_nodes` parameter.


.. _price_create_contract:

price_create_contract
---------------------

    Fuel cost for creating a new contract.

    This parameter defines additional fuel cost of the ``@1NewContract`` contract. When this contract is executed, fuel costs for executing functions in this contract are also counted and added to the total cost.

    This parameter is measured in fuel units. Fuel units are exchanged to |tokens| using :ref:`fuel_rate`.


.. _price_create_menu:

price_create_menu
-----------------

    Fuel cost for creating a new menu.

    This parameter defines additional fuel cost of the ``@1NewMenu`` contract. When this contract is executed, fuel costs for executing functions in this contract are also counted and added to the total cost.

    This parameter is measured in fuel units. Fuel units are exchanged to |tokens| using :ref:`fuel_rate`.


.. _price_create_page:

price_create_page
-----------------

    Fuel cost for creating a new page.

    This parameter defines additional fuel cost of the ``@1NewPage`` contract. When this contract is executed, fuel costs for functions in this contract are also counted and added to the total cost.

    This parameter is measured in fuel units. Fuel units are exchanged to |tokens| using :ref:`fuel_rate`.


.. _price_exec_address_to_id:

price_exec_address_to_id
------------------------

    Fuel cost of :func:`AddressToId` function call.


.. _price_exec_bind_wallet:

price_exec_bind_wallet
----------------------

    Fuel cost of :func:`Activate` function call.


.. _price_exec_column_condition:

price_exec_column_condition
---------------------------

    Fuel cost of :func:`ColumnCondition` function call.


.. _price_exec_compile_contract:

price_exec_compile_contract
---------------------------

    Fuel cost of :func:`CompileContract` function call.


.. _price_exec_contains:

price_exec_contains
-------------------

    Fuel cost of :func:`Contains` function call.


.. _price_exec_contract_by_id:

price_exec_contract_by_id
-------------------------

    Fuel cost of :func:`GetContractById` function call.


.. _price_exec_contract_by_name:

price_exec_contract_by_name
---------------------------

    Fuel cost of :func:`GetContractByName` function call.


.. _price_exec_contracts_list:

price_exec_contracts_list
-------------------------

    Fuel cost of :func:`ContractsList` function call.


.. _price_exec_create_column:

price_exec_create_column
------------------------

    Fuel cost of :func:`CreateColumn` function call.


.. _price_exec_create_ecosystem:

price_exec_create_ecosystem
---------------------------

    Fuel cost of :func:`CreateEcosystem` function call.


.. _price_exec_create_table:

price_exec_create_table
-----------------------

    Fuel cost of :func:`CreateTable` function call.


.. _price_exec_ecosys_param:

price_exec_ecosys_param
-----------------------

    Fuel cost of :func:`EcosysParam` function call.


.. _price_exec_eval:

price_exec_eval
---------------

    Fuel cost of :func:`Eval` function call.


.. _price_exec_eval_condition:

price_exec_eval_condition
-------------------------

    Fuel cost of :func:`EvalCondition` function call.


.. _price_exec_flush_contract:

price_exec_flush_contract
-------------------------

    Fuel cost of :func:`FlushContract` function call.


.. _price_exec_has_prefix:

price_exec_has_prefix
---------------------

    Fuel cost of :func:`HasPrefix` function call.


.. _price_exec_id_to_address:

price_exec_id_to_address
------------------------

    Fuel cost of :func:`IdToAddress` function call.


.. _price_exec_is_object:

price_exec_is_object
--------------------

    Fuel cost of :func:`IsObject` function call.


.. _price_exec_join:

price_exec_join
----------------

    Fuel cost of :func:`Join` function call.


.. _price_exec_json_to_map:

price_exec_json_to_map
----------------------

    Fuel cost of :func:`JSONToMap` function call.


.. _price_exec_len:

price_exec_len
--------------

    Fuel cost of :func:`Len` function call.


.. _price_exec_perm_column:

price_exec_perm_column
----------------------

    Fuel cost of :func:`PermColumn` function call.


.. _price_exec_perm_table:

price_exec_perm_table
---------------------

    Fuel cost of :func:`PermTable` function call.


.. _price_exec_pub_to_id:

price_exec_pub_to_id
--------------------

    Fuel cost of :func:`PubToID` function call.


.. _price_exec_replace:

price_exec_replace
------------------

    Fuel cost of :func:`Replace` function call.


.. _price_exec_sha256:

price_exec_sha256
-----------------

    Fuel cost of :func:`Sha256` function call.


.. _price_exec_size:

price_exec_size
---------------

    Fuel cost of :func:`Size` function call.


.. _price_exec_substr:

price_exec_substr
-----------------

    Fuel cost of :func:`Substr` function call.


.. _price_exec_sys_fuel:

price_exec_sys_fuel
-------------------

    Fuel cost of :func:`SysFuel` function call.


.. _price_exec_sys_param_int:

price_exec_sys_param_int
------------------------

    Fuel cost of :func:`SysParamInt` function call.


.. _price_exec_sys_param_string:

price_exec_sys_param_string
---------------------------

    Fuel cost of :func:`SysParamString` function call.


.. _price_exec_table_conditions:

price_exec_table_conditions
---------------------------

    Fuel cost of :func:`TableConditions` function call.


.. _price_exec_unbind_wallet:

price_exec_unbind_wallet
------------------------

    Fuel cost of :func:`Deactivate` function call.


.. _price_exec_update_lang:

price_exec_update_lang
----------------------

    Fuel cost of :func:`UpdateLang` function call.


.. _price_exec_validate_condition:

price_exec_validate_condition
-----------------------------

    Fuel cost of :func:`ValidateCondition` function call.


.. _price_tx_data:

price_tx_data
-------------

    Fuel cost taken per 1024 bytes of data passed to a transaction.

    This parameter is measured in fuel units.


.. _rollback_blocks:

rollback_blocks
---------------

    Number of blocks that can be rolled back in case when a fork is detected in the blockchain.
