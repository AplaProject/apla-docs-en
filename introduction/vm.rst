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

Below are the commands to work directly with the stack. The *Value* field is not used in them. It should be noted that now there is no fully automatic type conversion. For example, *string + float | int | decimal => float | int | decimal, float + int | str => float*, but *int + string => runtime error*.

* **cmdNot** – logic negation *(val) => (! ValueToBool (val))*,
* **cmdSign** – change of sign. *(val) => (-val)*,
* **cmdAdd** – adding. *(val1) (val2) => (val1 + val2)*,
* **cmdSub** – subtraction. *(val1) (val2) => (val1-val2)*,
* **cmdMul** – multiplication. *(val1) (val2) => (val1 * val2)*,
* **cmdDiv** – division. *(val1) (val2) => (val1 / val2)*,
* **cmdAnd** – logical AND *(val1) (val2) => (valueToBool (val1) && valueToBool (val2))*,
* **cmdOr** – logical OR. *(val1) (val2) => (valueToBool (val1) || valueToBool (val2))*,
* **cmdEqual** – equality comparison, bool is returned. *(val1) (val2) => (val1 == val2)*,
* **cmdNotEq** – comparison for inequality, bool is returned. *(val1) (val2) => (val1! = val2)*,
* **cmdLess** – comparison for being less, bool is returned. *(val1) (val2) => (val1 <val2)*
* **cmdNotLess** – the comparison for being greater or equal, bool is returned. *(val1) (val2) => (val1> = val2)*,
* **cmdGreat** – comparison for being greater, bool is returned. *(val1) (val2) => (val1> val2)*,
* **cmdNotGreat** – comparison for being less or equal, bool is returned. *(val1) (val2) => (val1 <= val2)*.

As already noted, the execution of bytecode does not affect the virtual machine. This, for example, allows you to simultaneously run various functions and contracts within a single virtual machine. The **Runtime** structure is used in order to start functions and contracts, as well as any expressions and bytecodes.

.. code:: 

    type RunTime struct {
       stack []interface{}
       blocks []*blockStack
       vars []interface{}
       extend *map[string]interface{}
       vm *VM
       cost int64
       err error
    }
    
* **stack** – the stack on which the bytecode is executed,
* **blocks** – block calls stack.

.. code:: 

    type blockStack struct {
         Block *Block
         Offset int
    }
    
* **Block** – indicator of the block being executed,
* **Offset** – the offset of the last command executed in the bytecode of the specified block,
* **vars** – stack of variables. When calling a bytecode in a block, its variables are added to this stack of variables.

After exiting the block, the size of the variables stack returns to the previous value.

* **extend** – a map indicator with values of external variables ($name),
* **vm** – a virtual machine indicator,
* **cost** – the resulting cost of execution,
* **err** – run error if occurred.

Running the bytecode occurs in the RunCode function. It contains a loop that performs the appropriate actions for each bytecode command. Before starting the bytecode processing, we must initialize the necessary data. Here we add our block to the

.. code:: 

    rt.blocks = append(rt.blocks, &blockStack{block, len(rt.vars)})
        
Next, we get information about the parameters of the "tail" functions, which should be in the last element of the stack.
    
.. code:: 

    var namemap map[string][]interface{}
    if block.Type == ObjFunc && block.Info.(*FuncInfo).Names != nil {
        if rt.stack[len(rt.stack)-1] != nil {
            namemap = rt.stack[len(rt.stack)-1].(map[string][]interface{})
        }
        rt.stack = rt.stack[:len(rt.stack)-1]
    }
    
Next, we must initialize all the variables that are defined in this block with initial values.

.. code:: 

   start := len(rt.stack)
   varoff := len(rt.vars)
   for vkey, vpar := range block.Vars {
      rt.cost--
      var value interface{}
      
Since our functions’ variables are also variables, we need to take them from the last elements of the stack in the same order as they are described in the function itself.

.. code:: 

   if block.Type == ObjFunc && vkey < len(block.Info.(*FuncInfo).Params) {
      value = rt.stack[start-len(block.Info.(*FuncInfo).Params)+vkey]
   } else {
   
Here we initialize local variables with initial values.

.. code:: 

        value = reflect.New(vpar).Elem().Interface()
        if vpar == reflect.TypeOf(map[string]interface{}{}) {
           value = make(map[string]interface{})
        } else if vpar == reflect.TypeOf([]interface{}{}) {
           value = make([]interface{}, 0, len(rt.vars)+1)
        }
     }
     rt.vars = append(rt.vars, value)
   }
   
Next, we need to update the values ​​of the variable parameters that were transferred in the "tail" functions.

.. code:: 

   if namemap != nil {
     for key, item := range namemap {
       params := (*block.Info.(*FuncInfo).Names)[key]
       for i, value := range item {
          if params.Variadic && i >= len(params.Params)-1 {
          
If it is possible to transfer a variable number of parameters, then we combine them into one variable array.

.. code:: 

                 off := varoff + params.Offset[len(params.Params)-1]
                 rt.vars[off] = append(rt.vars[off].([]interface{}), value)
             } else {
                 rt.vars[varoff+params.Offset[i]] = value
           }
        }
      }
   }
   
After that, all we are left to do is move the stack by removing the values that were transferred as parameters of the function from the stack top. We have already copied their values ​​into an array of variables.

.. code:: 

    if block.Type == ObjFunc {
         start -= len(block.Info.(*FuncInfo).Params)
    }
    
After the bytecode commands execution loop is over, we must correctly clear the stack.

.. code:: 

    last := rt.blocks[len(rt.blocks)-1]
    
Remove the current block from the stack of blocks.

.. code:: 

    rt.blocks = rt.blocks[:len(rt.blocks)-1]
    if status == statusReturn {

In case of a successful exit from the executed function, we add the return values ​​to the previous end of the stack.

.. code:: 

   if last.Block.Type == ObjFunc {
       for count := len(last.Block.Info.(*FuncInfo).Results); count > 0; count-- {
          rt.stack[start] = rt.stack[len(rt.stack)-count]
          start++
      }
     status = statusNormal
   } else {
   
As you can see, if that is not a function that we perform, then we do not restore the stack state, but we exit the function as is. The thing is that loops and conditional constructions already executed inside a function are also the bytecode block.

.. code:: 

        return
      }
    }
    rt.stack = rt.stack[:start]
    
Let us consider other functions for working with a virtual machine. Any virtual machine is created using the NewVM function. Three functions of **ExecContract**, **CallContract** and **Settings** are immediately added to each virtual machine. The adding occurs using the **Extend** function.

.. code:: 

   for key, item := range ext.Objects {
       fobj := reflect.ValueOf(item).Type()

We go through all the transferred objects and look only at the functions.
       
.. code:: 

   switch fobj.Kind() {
   case reflect.Func:
   
According to the information received about the function, we fill the **ExtFuncInfo** structure and add it to the top-level map Objects by its name.

.. code:: 

  data := ExtFuncInfo{key, make([]reflect.Type, fobj.NumIn()), make([]reflect.Type, fobj.NumOut()), 
     make([]string, fobj.NumIn()), fobj.IsVariadic(), item}
  for i := 0; i < fobj.NumIn(); i++ {
  
We have the so-called **Auto** parameters. Typically, this is the first parameter, for example sc *SmartContract* or rt *Runtime*. We cannot transfer them from the Simvolio language, but they are necessary for us when performing some golang functions. Therefore, we specify which variables will be automatically used at the time the function is called. In this case, the **ExecContract**, **CallContract** functions have such rt *Runtime* parameter. 

.. code:: 

  if isauto, ok := ext.AutoPars[fobj.In(i).String()]; ok {
     data.Auto[i] = isauto
  }
  
We fill in the information about the parameters

.. code:: 

    data.Params[i] = fobj.In(i)
  }
  
and about the types of returned values

.. code:: 

   for i := 0; i < fobj.NumOut(); i++ {
      data.Results[i] = fobj.Out(i)
   }
   
Adding a function to the root Objects will allow the compiler to find them later when used from contracts.

.. code:: 

             vm.Objects[key] = &ObjInfo{ObjExtFunc, data}
        }
    }
    
************************************************************
Compilation
************************************************************    
   
The functions located in the *compile.go* file are responsible for the compilation of the array of tokens obtained from the lexical analyzer. The compilation can be conditionally divided into two levels. At the top level, we process functions, contracts, blocks of code, conditional statements and loop statements, variable definitions, and so on. At the lower level, we compile expressions that are inside of code blocks or conditions in a loop and a conditional statement. In the beginning, let us consider a simpler lower level.
Translating expressions into a bytecode is done in the **compileEval** function. Since we have a virtual machine working with a stack, it is necessary to translate the usual infix record of expressions into a postfix notation or a reverse Polish notation. For example, 1 +2 should be converted to 12+, then you put 1 and 2 to the stack, and then we apply the addition operation for the last two elements in the stack and write the result to the stack. The translation algorithm itself can be found on the Internet – for example, https://master.virmandy.net/perevod-iz-infiksnoy-notatsii-v-postfiksnuyu-obratnaya-polskaya-zapis/. The global variable *opers = map [uint32] operPrior* contains the priorities of the operations that are necessary when translating into the reverse Polish notation. The following variables are defined at the beginning of the function:

* **buffer** – temporary buffer for bytecode commands,
* **bytecode** – final buffer of bytecode commands,
* **parcount** – temporary buffer for calculating parameters when calling functions,
* **setIndex** – the variable in the process of work is set to *true*, when we are assigning to the *map* or *array* element. For example, *a["my"] = 10*. In this case, we will need to use the special **cmdSetIndex command**.

Then there is a loop in which we get the next token and process it accordingly. For example, if braces are found
    
.. code:: 

    case isRCurly, isLCurly:
         i--
        break main
    case lexNewLine:
          if i > 0 && ((*lexems)[i-1].Type == isComma || (*lexems)[i-1].Type == lexOper) {
               continue main
          }
         for k := len(buffer) - 1; k >= 0; k-- {
              if buffer[k].Cmd == cmdSys {
                  continue main
             }
         }
        break main
        
we stop parsing the expression, and when moving the string, we look at whether the previous statement is an operation and whether we are inside the parentheses, otherwise we exit and the expression is parsed. In general, the algorithm itself corresponds to an algorithm for translating into a reverse Polish notation, taking into account that it is necessary to take the calls of functions, contracts, index calls, and other things that you will not meet in case of parsing, for example, for a calculator, into account. Consider the option of parsing the *lexIdent* type token. We are looking for a variable, function or contract with this name. If nothing is found and this is not a function or a contract call, then we indicate an error.

.. code:: 

    objInfo, tobj := vm.findObj(lexem.Value.(string), block)
    if objInfo == nil && (!vm.Extern || i > *ind || i >= len(*lexems)-2 || (*lexems)[i+1].Type != isLPar) {
          return fmt.Errorf(`unknown identifier %s`, lexem.Value.(string))
    }
    
We may have a situation where a contract called will be described later. In this case, if a function and a variable with the same name are not found, then we believe that we will have a contract call. In a language, the contracts and functions calls do not differ. But we need to call the contract through the **ExecContract** function, the one we use in the bytecode.
    
 .. code:: 

    if objInfo.Type == ObjContract {
        objInfo, tobj = vm.findObj(`ExecContract`, block)
        isContract = true
    }
    
In *count*, we will write down the number of variables so far and this value will also go to the stack with the number of function parameters. We simply increase this quantity by one unit in the last element of the stack at each subsequent detection of the parameter.

.. code:: 

    count := 0
    if (*lexems)[i+2].Type != isRPar {
        count++
    }
    
Since we have the *Used* list of called parameters for contracts, then we need to make the marks for the case of contract being called, and in case the contract is called without the *MyContract()* parameters, we have to add two empty parameters to the call **ExecContract**, which should get the minimum two parameters.
 
.. code:: 

    if isContract {
       name := StateName((*block)[0].Info.(uint32), lexem.Value.(string))
       for j := len(*block) - 1; j >= 0; j-- {
          topblock := (*block)[j]
          if topblock.Type == ObjContract {
                if topblock.Info.(*ContractInfo).Used == nil {
                     topblock.Info.(*ContractInfo).Used = make(map[string]bool)
                }
               topblock.Info.(*ContractInfo).Used[name] = true
           }
        }
        bytecode = append(bytecode, &ByteCode{cmdPush, name})
        if count == 0 {
           count = 2
           bytecode = append(bytecode, &ByteCode{cmdPush, ""})
           bytecode = append(bytecode, &ByteCode{cmdPush, ""})
         }
        count++

    }
    
If we see that there is a square bracket next, then we add the **cmdIndex** command to get the value by the index.

.. code:: 

    if (*lexems)[i+1].Type == isLBrack {
         if objInfo == nil || objInfo.Type != ObjVar {
             return fmt.Errorf(`unknown variable %s`, lexem.Value.(string))
         }
        buffer = append(buffer, &ByteCode{cmdIndex, 0})
    }
    
The **compileEval** function generates the bytecode of the expressions in blocks directly, but the **CompileBlock** function forms both the object tree and the bytecode not related to the expressions. Compilation is also based on the work of the finite state machine, just as it was done for lexical analysis, but with the following differences. First, we do not operate with symbols but with tokens, and second, we describe all states and transitions in *states* variable immediately. It represents an array of maps with indices by type of tokens and each token has the structure of the *compileState* with a new state specified in the *NewState*, and in case it is clear what structure we have parsed, then the function of the handler in the *Func* field is specified.

Let us review the main state as an example
  
.. code:: 

    { // stateRoot
       lexNewLine: {stateRoot, 0},
       lexKeyword | (keyContract << 8): {stateContract | statePush, 0},
       lexKeyword | (keyFunc << 8): {stateFunc | statePush, 0},
       lexComment: {stateRoot, 0},
       0: {errUnknownCmd, cfError},
    },
    
If we encounter line break or comments, then we stay in the same state. If we encounter the **contract** keyword, then we change the state to the *stateContract* and begin parsing this construction. If we encounter the **func** keyword, then we change to the *stateFunc state*. If other tokens are received, the error generation function will be called. Suppose that we have encountered the *func* keyword and we have changed the state to *stateFunc*. 

.. code:: 

    { // stateFunc
        lexNewLine: {stateFunc, 0},
        lexIdent: {stateFParams, cfNameBlock},
        0: {errMustName, cfError},
    },
    
Since the name of the function must follow the **func** keyword, then when changing the string, we remain in the same state, and with all the other tokens we generate the corresponding error. If we get the function name in the token-identifier, then we go to *stateFParams* state in which we get the parameters of the function. In doing so, we call the **fNameBlock** function. It should be noted that the *Block* structure was created using the *statePush* flag, and here we take it from the buffer and fill with the data we need.

.. code:: 

    func fNameBlock(buf *[]*Block, state int, lexem *Lexem) error {
        var itype int

        prev := (*buf)[len(*buf)-2]
        fblock := (*buf)[len(*buf)-1]
       name := lexem.Value.(string)
       switch state {
         case stateBlock:
            itype = ObjContract
           name = StateName((*buf)[0].Info.(uint32), name)
           fblock.Info = &ContractInfo{ID: uint32(len(prev.Children) - 1), Name: name,
               Owner: (*buf)[0].Owner}
        default:
           itype = ObjFunc
           fblock.Info = &FuncInfo{}
         }
         fblock.Type = itype
        prev.Objects[name] = &ObjInfo{Type: itype, Value: fblock}
        return nil
    }
    
The **fNameBlock** function is used for contracts and functions (including those nested in other functions and contracts). It fills the *Info* field with the appropriate structure and writes itself into the map *Objects* of the parent block. This is done so that we can then call this function or contract by the given name. Similarly, we create functions for all states and variants. These functions are usually very small and perform some work on the formation of the virtual machine tree. As for the **CompileBlock** function, it simply goes through all the tokens and switches states according to those described in the *states*. Almost the whole additional processing code for additional flags.
    
* **statePush** – the *Block* object is added to the object tree,
* **statePop** – used when the block ends with closing curly braces,
* **stateStay** – indicates that when you change to a new state, you need to stay on the current token,
* **stateToBlock** – indicates the transition to *stateBlock* state. Used to handle while and if, when it is needed to go into * * **the processing** of the block inside curly brackets after the expression is processed,
* **stateToBody** – indicates the transition to *stateBody*,
* **stateFork** – saves the position of the token. Used when an expression starts in an identifier or a name with **$**. We can have either a function call or an assignment,
* **stateToFork** – used to get the token stored in the *stateFork* flag. This token will be passed to the processing function,
* **stateLabel** – serves for inserting the **cmdLabel** command. This flag is needed for the while construction,
* **stateMustEval** – checks for a conditional expression availability at the beginning of the if and while constructions.
    
Besides the **CompileBlock** function, you should also mention the **FlushBlock** function. The matter is that the tree of blocks is built independent of the existing virtual machine. More precisely, we take information about the functions and contracts existing in a virtual machine, but we gather the compiled blocks into a separate tree. Otherwise, if an error occurs during compilation, we will have to roll back the state of the virtual machine to the previous state. Therefore, we compile the tree separately, but have to call the **FlushContract** function after the compilation is successful. This function adds our finished block tree to the current virtual machine. At this point, the compilation stage is considered complete.
  
*******************************************************************
Lexical analysis
*******************************************************************    




    















