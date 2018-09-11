Platform parameters
===================

.. todo::

    Topics
        how these parameters are changed (voting)

    Reference 
        what each parameter does (system_parameters table contents)
        where these parameters are stored (system tables)



Platform parameters are configuration parameters for the blockchain platform. These parameters are stored in the ``system_parameters`` table. This table is avaliable in the 


Changing platform parameters
----------------------------
 
Changes to platform parameters must be introduced as a result of voting.

.. todo::

    Link to a topic where this is explained further.


System_parameters table
--------------------------

Ecosystem parameters are stored in the ``system_parameters`` table. This table is available in the default ecosystem that is created on the blockchain network.


Parameters by purpose
---------------------

Blockchain network
""""""""""""""""""

:ref:`full_nodes`
:ref:`number_of_nodes`
:ref:`incorrect_blocks_per_day`
:ref:`node_ban_time`
:ref:`local_node_ban_time`


User interfaces
"""""""""""""""

:ref:`default_ecosystem_page`

.. todo::

    Following params have no description

:ref:`default_ecosystem_menu`
:ref:`default_ecosystem_contract`



Database
""""""""

:ref:`max_columns`
:ref:`max_indexes`

Block generation
""""""""""""""""

:ref:`block_reward`
:ref:`gap_between_blocks`
:ref:`max_tx_size`
:ref:`max_tx_count`
:ref:`max_block_generation_time`
:ref:`max_block_user_tx`
:ref:`commission_wallet`
:ref:`commission_size`
:ref:`rb_blocks_1`
:ref:`blockchain_url`
:ref:`max_block_size`
:ref:`max_forsign_size`

.. todo::

    Following params have no description

:ref:`new_version_url`


Fuel
""""

:ref:`fuel_rate`
:ref:`max_fuel_block`
:ref:`max_fuel_tx`
:ref:`size_fuel`

:ref:`ecosystem_price`

.. todo::

    Following params have no description

:ref:`column_price`
:ref:`contract_price`
:ref:`menu_price`
:ref:`page_price`
:ref:`table_price`
:ref:`extend_cost_activate`
:ref:`extend_cost_address_to_id`
:ref:`extend_cost_column_condition`
:ref:`extend_cost_compile_contract`
:ref:`extend_cost_contains`
:ref:`extend_cost_contracts_list`
:ref:`extend_cost_create_column`
:ref:`extend_cost_create_ecosystem`
:ref:`extend_cost_create_table`
:ref:`extend_cost_deactivate`
:ref:`extend_cost_ecosys_param`
:ref:`extend_cost_eval`
:ref:`extend_cost_eval_condition`
:ref:`extend_cost_flush_contract`
:ref:`extend_cost_has_prefix`
:ref:`extend_cost_id_to_address`
:ref:`extend_cost_is_object`
:ref:`extend_cost_join`
:ref:`extend_cost_json_to_map`
:ref:`extend_cost_len`
:ref:`extend_cost_new_state`
:ref:`extend_cost_perm_column`
:ref:`extend_cost_perm_table`
:ref:`extend_cost_pub_to_id`
:ref:`extend_cost_replace`
:ref:`extend_cost_sha256`
:ref:`extend_cost_size`
:ref:`extend_cost_substr`
:ref:`extend_cost_sys_fuel`
:ref:`extend_cost_sys_param_int`
:ref:`extend_cost_sys_param_string`
:ref:`extend_cost_table_conditions`
:ref:`extend_cost_update_lang`
:ref:`extend_cost_validate_condition`



Parameters by name
------------------

.. _block_reward:

block_reward
""""""""""""

    // BlockReward value of reward, which is chrged on block generation


.. _blockchain_url:

blockchain_url
""""""""""""""
    
    url, which the last version of blockchain file can be downloaded

    // BlockchainURL is the address of the blockchain file.  For those who don't want to collect it from nodes

.. _column_price:

column_price
""""""""""""

    TBD

.. _commission_size:

commission_size
"""""""""""""""

    // CommissionSize is the value of the commission


.. _commission_wallet:

commission_wallet
"""""""""""""""""

    // CommissionWallet is the address for commissions

.. _contract_price:

contract_price
""""""""""""""

    TBD

.. _default_ecosystem_contract:

default_ecosystem_contract
""""""""""""""""""""""""""

    TBD

.. _default_ecosystem_menu:

default_ecosystem_menu
""""""""""""""""""""""

    TBD

.. _default_ecosystem_page:

default_ecosystem_page
""""""""""""""""""""""

    Source code of the default page for the default ecosystem.

.. _ecosystem_price:

ecosystem_price
"""""""""""""""

    Fuel cost for creating a new ecosystem.


.. _extend_cost_activate:

extend_cost_activate
""""""""""""""""""""
    TBD

.. _extend_cost_address_to_id:

extend_cost_address_to_id
"""""""""""""""""""""""""
    TBD

.. _extend_cost_column_condition:

extend_cost_column_condition
""""""""""""""""""""""""""""
    TBD

.. _extend_cost_compile_contract:

extend_cost_compile_contract
""""""""""""""""""""""""""""
    TBD

.. _extend_cost_contains:

extend_cost_contains
""""""""""""""""""""
    TBD

.. _extend_cost_contracts_list:

extend_cost_contracts_list
""""""""""""""""""""""""""
    TBD

.. _extend_cost_create_column:

extend_cost_create_column
"""""""""""""""""""""""""
    TBD

.. _extend_cost_create_ecosystem:

extend_cost_create_ecosystem
""""""""""""""""""""""""""""
    TBD

.. _extend_cost_create_table:

extend_cost_create_table
""""""""""""""""""""""""
    TBD

.. _extend_cost_deactivate:

extend_cost_deactivate
""""""""""""""""""""""
    TBD

.. _extend_cost_ecosys_param:

extend_cost_ecosys_param
""""""""""""""""""""""""
    TBD

.. _extend_cost_eval:

extend_cost_eval
""""""""""""""""
    TBD

.. _extend_cost_eval_condition:

extend_cost_eval_condition
""""""""""""""""""""""""""
    TBD

.. _extend_cost_flush_contract:

extend_cost_flush_contract
""""""""""""""""""""""""""
    TBD

.. _extend_cost_has_prefix:

extend_cost_has_prefix
""""""""""""""""""""""
    TBD

.. _extend_cost_id_to_address:

extend_cost_id_to_address
"""""""""""""""""""""""""
    TBD

.. _extend_cost_is_object:

extend_cost_is_object
"""""""""""""""""""""
    TBD

.. _extend_cost_join:

extend_cost_join
""""""""""""""""
    TBD

.. _extend_cost_json_to_map:

extend_cost_json_to_map
"""""""""""""""""""""""
    TBD

.. _extend_cost_len:

extend_cost_len
"""""""""""""""
    TBD

.. _extend_cost_new_state:

extend_cost_new_state
"""""""""""""""""""""
    TBD

.. _extend_cost_perm_column:

extend_cost_perm_column
"""""""""""""""""""""""
    TBD

.. _extend_cost_perm_table:

extend_cost_perm_table
""""""""""""""""""""""
    TBD

.. _extend_cost_pub_to_id:

extend_cost_pub_to_id
"""""""""""""""""""""
    TBD

.. _extend_cost_replace:

extend_cost_replace
"""""""""""""""""""
    TBD

.. _extend_cost_sha256:

extend_cost_sha256
""""""""""""""""""
    TBD

.. _extend_cost_size:

extend_cost_size
""""""""""""""""
    TBD

.. _extend_cost_substr:

extend_cost_substr
""""""""""""""""""
    TBD

.. _extend_cost_sys_fuel:

extend_cost_sys_fuel
""""""""""""""""""""
    TBD

.. _extend_cost_sys_param_int:

extend_cost_sys_param_int
"""""""""""""""""""""""""
    TBD

.. _extend_cost_sys_param_string:

extend_cost_sys_param_string
""""""""""""""""""""""""""""
    TBD

.. _extend_cost_table_conditions:

extend_cost_table_conditions
""""""""""""""""""""""""""""
    TBD

.. _extend_cost_update_lang:

extend_cost_update_lang
"""""""""""""""""""""""
    TBD

.. _extend_cost_validate_condition:

extend_cost_validate_condition
""""""""""""""""""""""""""""""
    TBD


.. _fuel_rate:

fuel_rate
"""""""""

    // FuelRate is the rate

.. _full_nodes:

full_nodes
""""""""""
    
    List of validating nodes.

    Format:

        ``[["host1:port","-1222","nodepub1"], ["host2:ip","-1222","nodepub2"]]``

        where

        ``host1:port`` is the address of the host where transactions and new blocks are sent; this address can also be used to receive the full blockchain starting from the first block.

        ``-1222`` is the account identifier that receives the commission; if this account oes not exist, the commission is not collected.

        ``nodepub1`` public key of the node; this key is required to check block signatures.


.. _gap_between_blocks:

gap_between_blocks
""""""""""""""""""

    Amount of time, in seconds, allotted to the validating node to create a new block.

    Minimum value for this parameter is ``1`` (one second).

.. _incorrect_blocks_per_day:

incorrect_blocks_per_day
""""""""""""""""""""""""

    // IncorrectBlocksPerDay is value of incorrect blocks per day before global ban

.. _local_node_ban_time:

local_node_ban_time
"""""""""""""""""""

    // LocalNodeBanTime is value of local ban time for bad nodes (in ms)

.. _max_block_generation_time:

max_block_generation_time
"""""""""""""""""""""""""
    
    // MaxBlockGenerationTime is the time limit for block generation (in ms)


.. _max_block_size:

max_block_size
""""""""""""""

    Maximum block size.

    .. todo::

        Size in bytes?

.. _max_block_user_tx:

max_block_user_tx
"""""""""""""""""

    // MaxBlockUserTx is the maximum number of user's transactions in one block

.. _max_columns:

max_columns
"""""""""""
    // MaxColumns is the maximum columns in tables

.. _max_forsign_size:

max_forsign_size
""""""""""""""""

    // MaxForsignSize is the maximum size of the forsign of transaction

.. _max_fuel_block:

max_fuel_block
""""""""""""""

    // MaxBlockFuel is the maximum fuel of the block


.. _max_fuel_tx:

max_fuel_tx
"""""""""""

    // MaxTxFuel is the maximum fuel of the transaction

.. _max_indexes:

max_indexes
"""""""""""

    // MaxIndexes is the maximum indexes in tables

.. _max_tx_count:

max_tx_count
""""""""""""

    // MaxTxCount is the maximum count of the transactions

.. _max_tx_size:

max_tx_size
"""""""""""

    // MaxTxSize is the maximum size of the transaction

.. _menu_price:

menu_price
""""""""""

    TBD

.. _new_version_url:

new_version_url
"""""""""""""""

    TBD


.. _node_ban_time:

node_ban_time
"""""""""""""

    // NodeBanTime is value of ban time for bad nodes (in ms)

.. _number_of_nodes:

number_of_nodes
"""""""""""""""

    Maximum number of validating nodes in :ref:`full_nodes` parameter.

.. _page_price:

page_price
""""""""""

    TBD

.. _rb_blocks_1:

rb_blocks_1
"""""""""""

    // RbBlocks1 rollback from queue_bocks

.. _size_fuel:

size_fuel
"""""""""

    // SizeFuel is the fuel cost of 1024 bytes of the transaction data

    .. todo::

        found in source, not in product

.. _table_price:

table_price
"""""""""""

    TBD