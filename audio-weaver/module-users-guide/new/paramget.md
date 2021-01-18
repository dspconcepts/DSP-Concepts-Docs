---
description: >-
  Gets the value of a module variable from another module and writes this to its
  output wire.
---

# ParamGet

The ParamGet module has no input pins and one output pin. ParamGet gets a module variable from another module and outputs this variable on its output pin. ParamGet outputs can be used as control variables for other modules.

The ParamGet module gets the value of a module variable from another module and writes this to its output wire. This allows the variables to be used for other calculations, or simply to be monitored with a sink display.

At instantiation time \(or in Module Name and Arguments in Audio Weaver Designer\), you specify the module variable to be fetched and its data type.

The variable is specified as

`Mod.Var`

where `Mod` is the name of the module \(in the same level of the hierarchy as the ParamGet module\) and `Var` is the variable name. You can also specify modules that are within subsystems using

`Subsystem.Mod.Var`

You can also go up in hierarchy and get variables from modules that are in an upper system. Use a single backslash character to move up one level in the hierarchy. Use multiple backslash characters to traverse multiple levels of hierarchy.

### Arguments

* `dataType` - Data type of the variable in the targeted module. Note that the data type selected must match the data type of the variable in the targeted module or an error will be thrown.
* `modVar` - Specifies the internal variable to get in module.variable. Please note that you can only get module variables and not module arguments. If you try to get an argument the first variable of the target module is returned instead.

## Examples

### Getting Variables From a Module in a Different Level

Suppose that the ParamGet module is located in:

`SYS.Subsystem.ParamGet`

and you want to get the variable

`SYS.Scaler1.gain`

Then set the `Mod.Var` field to

`\Scaler1.gain`

If the variable is an array you can also get individual values from arrays using the syntax

`Mod.Var[index]`

where `index` is the zero-based index of the element in the array `Var` to get.

### Display Variable From Sine Module

Suppose your signal flow has a sine wave and you wish to use the frequency of the sine wave in a different part of your signal flow or maybe you just want to display the frequency.

The figure below shows how ParamGet can retrieve the `freq` variable from Sine1 and display it. The ParamGet argument are set to:

![](../../../.gitbook/assets/0%20%2837%29.png)

![](../../../.gitbook/assets/1%20%2836%29.png)

## Inspector

This module does not have an inspector.

## Matlab Usage

```
M=param_get_module(NAME, DATATYPE, MODVAR)
```

This module can reach into the instance structure of other modules and gets the parameter values.

### Arguments

* `NAME` - name of the module.
* `DATATYPE` - string specifying the data type of the variable and this is also used for the data type of the output pin. Allowable values are 'float', 'int', or 'fract32'.
* `MODVAR` - specifies the module and variable name using the form:
  * '`MOD.VAR`' where `MOD` is the module name and `VAR` is the variable name.
  * You can also specify internal subsystems using '`SUBSYS.MOD.VAR`'.
  * Module variables at upper levels of hierarchy can be accessed as well using the \ operator. E.g. `\MOD.VAR (1 level up)` `\\MOD.VAR (2 levels up)` `Etc.` 

## Type Definition

```cpp
typedef struct _ModuleParamGet
{
    ModuleInstanceDescriptor instance;  // Common Audio Weaver module instance
                                        // structure.
    void * modPtr; // Points to the module to get
    void * varPtr; // Points to the variable to get within the
            // module instance structure
} ModuleParamGetClass;
```

## See Also

 ParamSet, PresetArrayHandler, SetWireProperties, StatusSet, UpdateSampleRate

