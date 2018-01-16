################################################################################
Compiler and virtual machine
################################################################################

This section reviews the code, organization and code operation in the packages/script directory, which refers to the compilation and operation of the Simvolio program language virtual machine. The documentation is intended primarily for developers.

Working with contracts is organized as follows: contracts and functions are written using the Simvolio language and stored in the contracts tables in ecosystems. When the program starts, it reads the source code from the database and compiles it into a bytecode. When adding or changing contracts and writing them to the blockchain, the updated data is compiled and the corresponding virtual machine bytecode is added/updated. The bytecode is not physically saved anywhere, so when you exit the program and start again, the compilation happens anew. A virtual machine is a set of objects – contracts, functions, types, etc. The entire source code described in the contracts table for all ecosystems is compiled in strict sequence into one virtual machine and the state of the virtual machine is the same on all nodes. When you call a contract, the virtual machine does not change its state in any way. Any execution of a contract or a function call occurs on a separate runtime stack that is created with each external call. Each ecosystem can have a so-called virtual ecosystem that works with its tables outside the blockchain, within one node, and cannot directly affect the blockchain or other virtual ecosystems. In this case, a node hosting such a virtual ecosystem compiles its contracts and creates the virtual machine for it.

********************************************************************************
Virtual machine
********************************************************************************
Let us review how the virtual machine is organized in memory.

.. code:: 

    type VM struct {
       Block         
       ExtCost func(string) int64
       FuncCallsDB map[string]struct{}
       Extern bool 
    }
    
* **Block** - is the most important structure that contains all the information,
* **ExtCost** - is a function that returns the cost of executing external golang functions,
* **FuncCallsDB** – map of the names of golang functions’ that return the cost of execution to the first parameter. These are the functions of working with databases that calculate the cost using EXPLAIN,
* **Extern** – when you create a VM, it is set to true and does not require the contract called to be present when compiling the code. That is, it allows calling the contract, which will be determined in the future.

A virtual machine is a tree of **Block** type objects. In fact, a block is an independent unit containing some bytecode. In simple words – everything put in braces in the language is a block. For example,

.. code:: 

    func my() {
         if true {
              while false {
                   ...
               }
         }
    } 
    
    creates the block with a function in which there is a block with if having, in turn, a block with while.
    
    .. code:: 

    type Block struct {
        Objects map[string]*ObjInfo
        Type int
        Owner *OwnerInfo
        Info interface{}
        Parent *Block
        Vars []reflect.Type
        Code ByteCodes
        Children Blocks
    }
    
* **Objects** – map of internal objects of **ObjInfo** indicator type. If, for example, there is a variable in the block, then we can quickly get information about it by name,
* **Type** – the type of the block, functions and contracts equal to **ObjFunc** and **ObjContract**,
* **Owner** – a link to the **OwnerInfo** structure, which contains information about the owner of the compiled contract. It is specified during compilation of contracts or is loaded from the **contracts** table,
* **Info** – contains information about the object and depends on the block type,
* **Parent** – parent block indicator,
* **Vars** – an array with the current block variables types,
* **Code** – the bytecode itself, which will be executed when the control is transferred to this block. For example, in the case of calling the function or the loop body,
* **Children** – an array of child blocks. For example, nested functions, loops, conditional operators,

Let us review another important structure of **ObjInfo**.

.. code:: 

    type ObjInfo struct {
       Type int
       Value interface{}
    }
    
**Type** – the object type can take one of the following values:

* **ObjContract** – contract,
* **ObjFunc** – function,
* **ObjExtFunc** – external golang function,
* **ObjVar** – variable,
* **ObjExtend** – the $name variable.

**Value** – contains the appropriate structure for each type.

.. code:: 

    type ContractInfo struct {
        ID uint32
        Name string
        Owner *OwnerInfo
        Used map[string]bool
        Tx *[]*FieldInfo
        Settings map[string]interface{}
    }
    
* **ID** – contract identifier. This value is indicated in the blockchain when calling the contract,
* **Name** – contract name,
* **Owner** – additional information about the contract,
* **Used** – map of the names of contracts called inside,
* **Tx** – data array described in the data section of the contract.

.. code:: 

    type FieldInfo struct {
           Name string
          Type reflect.Type
          Tags string
    }
    
where **Name** is the name of the field, **Type** is the type, **Tags** – additional tags for the field.

**Settings** – map of the values ​​that are described in the settings section of the contract.

As you can see, the information is largely duplicated with the block structure. This can be considered an architectural drawback, from which it is desirable to get rid of.

For the **ObjFunc** type, the **Value** field contains the **FuncInfo** structure.

.. code:: 

    type FuncInfo struct {
         Params []reflect.Type
         Results []reflect.Type
        Names *map[string]FuncName
        Variadic bool
        ID uint32
    }
    
* **Params** – an array of parameter types,
* **Results** – an array of returned types,
* **Names** – map for tail functions data. For example, ``DBFind().Columns ()``.

.. code:: 

    type FuncName struct {
       Params []reflect.Type
       Offset []int
       Variadic bool
    }
    
* **Params** – an array of parameter types,
* **Offset** – an array of offsets for these variables. In fact, all parameters that are expressed in functions using the dot are variables that can be assigned initialization values,
* **Variadic** – true, if the tail description can have the number of parameters as a variable.

* **Variadic** – true if the function can the number of parameters as a variable,
* **ID** – function identifier.

For the **ObjExtFunc type**, the **Value** field contains the structure of **ExtFuncInfo**. It describes the functions on golang.

.. code:: 

    type ExtFuncInfo struct {
       Name string
       Params []reflect.Type
       Results []reflect.Type
       Auto []string
       Variadic bool
       Func interface{}
    }
    
The matching parameters are the same as for the **FuncInfo** structure. 
**Auto** – an array of variables that are additionally passed to the golang functions, if any. For example, the sc variables of *SmartContract* type, 
**Func** – golang function.

**For** - the **ObjVar** type, the **Value** field contains a **VarInfo** structure.

.. code:: 

    type VarInfo struct {
       Obj *ObjInfo
       Owner *Block
    }

* **ObjInfo** – information about the type and value of the variable,
* **Owner** – owner block indicator.

For **ObjExtend** objects, the **Value** field contains a string with the name of the variable or the function.

Virtual machine commands
============================

The identifiers of the virtual machine commands are described in the *cmds_list.go file*. The bytecode is a sequence of **ByteCode** type structures.

.. code:: 

    type ByteCode struct {
       Cmd uint16
       Value interface{}
    }

The **Cmd** field stores the command identifier, and the **Value** field contains the supporting value. As a rule, commands perform operations to the final elements of the stack, and write the resulting value there if necessary.

* **cmdPush** – place a value from the *Value* field to the stack. For example, it is used to put numbers and lines to the stack,
* **cmdVar** – put the value of the variable to the stack. *Value* contains an indicator of the *VarInfo* structure and the information about the variable,
* **cmdExtend** – put the value of an external variable to the stack, they start with **$**. *Value* contains a string with the variable name,
* **cmdCallExtend** – call the external function, their names begin with **$**. The parameters of the function will be taken from the stack, and the result(s) of the function will be placed to the stack. Value contains the name of the function,
* **cmdPushStr** – place the string from *Value* to the stack,
* **cmdCall** – call the virtual machine function. *Value* contains the **ObjInfo** structure. This command is applicable both for *ObjExtFunc* golang and for *ObjFunc* Simvolio functions. When a function is called, the transferred parameters are taken from the stack, and the resulting values are returned to the stack,
* **cmdCallVari** – similarly to the **cmdCall** command calls the virtual machine function, but this command is used to call functions with a variable number of parameters,
* **cmdReturn** – is used to exit the function. The returned values are placed to the stack. *Value* is not used,
* **cmdIf** – transfers control to the bytecode in the **Block** structure, an indicator to which is transferred to the *Value* field. Control is only transferred if calling the *valueToBool* function with the edge stack element returns *true*. Otherwise, control is transferred to the next command,
* **cmdElse** – the command works in the same way as the **cmdIf** command, but the control is transferred to the specified block only if *valueToBool* with the edge stack element returns false.
* **cmdAssignVar** – gets the list of **VarInfo** variables from *Value*, which get a value with the cmdAssign command,
* **cmdAssign** – assign to variables obtained by the **cmdAssignVar** command the values from the stack,
* **cmdLabel** – defines a label where control will be returned to during the while loop,
* **cmdContinue** – the command passes control to the **cmdLabel** label. Performs a new iteration of the loop. Value is not used,
* **cmdWhile** – checks the extreme element of the stack with *valueToBool* and calls the **Block** passed to the Value field, if the value is true,
* **cmdBreak** – exits the loop,
* **cmdIndex** – obtaining the *map* or *array* on the index value to the stack. *Value* is not used. *(map | array) (index value) => (map | array [index value])*,
* **cmdSetIndex** – assign the edge value of the stack to the map or array element. *Value* is not used. *(map | array) (index value) (value) => (map | array)*,
* **cmdFuncName** – adds parameters that are transferred using sequential descriptions divided by the dot *func name Func (...) .Name (...)*,
* **cmdError** – a command is created that terminates a contract or function with an error that was specified in *error, warning* or *info*.

    















