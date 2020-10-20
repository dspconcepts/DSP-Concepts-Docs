---
description: Sets variables in another module using control signals.
---

# ParamSet

The ParamSet module takes a signal from an input wire and writes the value to an instance variable in another module. The module is useful for controlling the behavior of other modules in response to control signals or audio signals.

At instantiation time \(or in Module Name and Arguments in Audio Weaver Designer\), you specify the data type of the input pin and also the internal variable which will be written. The internal variable is specified as:

`Mod.Var`

where `Mod` is the name of the module \(in the same level of the hierarchy as the ParamSet module\) and `Var` is the internal variable name. You can also specify modules that are within subsystems using

`Subsystem.Mod.Var`

You can also go up in hierarchy and control modules that are in an upper system. Use a single backslash character to move up one level in the hierarchy. Use multiple backslash characters to traverse multiple levels of hierarchy. Suppose that the ParamSet module is located in:

`SYS.Subsystem.ParamSet`

and you want to control the variable

`SYS.Scaler1.gain`

Then set the Mod.Var field to

`\Scaler1.gain`

And finally, you can also write to individual values in arrays using the syntax

`Mod.Var[index]`

where `index` is the zero-based index of the element in the array `Var` to set.

The variable `setBehavior` determines how often the internal variable is updated and whether the module's set function is called.

The input wire can have any number of channels and any block size The module only uses the first value in the wire and ignores all others.

Internally the module uses two pointers. The first points to the base of the module being controlled. The second pointer points to the internal variable being written.

### Arguments

![](../../../.gitbook/assets/0%20%2835%29.png)

* `dataType` - Data type of internal variable. The default is float. This type must match the type of the internal variable that modVar points to and the data type of the input wire.
* `modVar` - Specifies the internal marible to set in module.variable format or subsystem.module.variable. See above description to see format options.

### Variables

![](../../../.gitbook/assets/1%20%2829%29.png)

`setBehavior` has 5 modes. The Matlab interface uses the enumerated value while the Designer variable is a string:

* \(0,AlwaysNoSet\) = Always update. Do not call a set function \(useful\).
* \(1,AlwaysDeferredSet\) = Always update. Call set function \(not too useful\)
* \(2,OnChangeNoSet\) = Update only when input value changes. Do not call a set function \(useful\).
* \(3,OnChangeDeferredSet\) = Update only when input value changes. Call set function \(Most useful\).
* \(4,OnChangeInstantSet\) = Update only when input value changes. Call set function immediately \(not deferred\).

Note that in cases 1 and 3 the set function is triggered by setting the deferred bit in the module instance structure which causes the module's set function to be called as a background task. It may take tens of milliseconds before the set function is actually called and the exact timing can depend upon the CPU loading of the system. If the system is lightly loaded then there is more CPU power available for performing the deferred processing tasks. For case 4, the set function is called immediately in the real-time thread. Use with caution since this increases the CPU load.

## Examples

### Remote Control of Balance

In the signal flow below let’s assume that the balance module needs to be controlled by someone driving a car. When the driver turns the balance knob a processor in the head unit reads the knob position and sends the results to the digital signal processor \(DSP\) where Audio Weaver is running. The subsystem called BalanceControlInterface represents the interface between the head unit and the DSP. You will notice that the line between the interface and the ParamSet module is dashed. This indicates that it is a control wire with a block size of 1. Since the Balance module does not have an input for the `balance` variable, ParamSet is used to modify the internal variable.

![](../../../.gitbook/assets/2%20%2833%29.png)

### Examples of Different setBehavior Values

Before we jump into this example we need to define a few things. A set function is a C function that is called in a module when a variable changes. The set function is useful when an internal variable is calculated from an input variable. For example the Balance Module uses a panning law to calculate the gains for the left and right channels. Having said this, it should be pointed out that not all modules need a set function. When a variable is changed it is immediately used by the module.

* AlwaysNoSet - This behavior is interpreted as follows: Always means that the balance variable in the above example would always be updated at the beginning of each block. NoSet means that the set function is never called.  So for the example above if we start with the balance command at 0 and then try to move it to one.  The Balance module does not change its gains because the set function is never called.
  * ![](https://lh5.googleusercontent.com/MZQQUrEhkyqGCJ1H-gpCY0Z3YcpYgqMAs8yKAA3xofGnDLRrFHLcSMWaUEyubsldl2o4YRfShz5wjVup1wZyFkTcgVG2z5DVJ_oM_ffOZnWgfexJRZ5V2grIPsPp_2lJ1avFrK42) The gains remain in the center position.
* AlwaysDefferedSet - Always again means that the balance variable is transmitted from paramSet to the balance variable at the beginning of each buffer.  DefferedSet means that the set function will be processed at low priority in the background. Now when balance is changed to 1 the set function is called.
  * ![](https://lh3.googleusercontent.com/c99trkiMcgx8M06zr35VTpvblFWY8iF7SFJgefAq_qM284tBsmsUdsG2mwulcE8VqhFh-UbzlZfCw1Ktd60fBZjhK07YjxSt1dGsaY8_gh-83_qDnW4EjPqBK6skfLhyAbkCC4as) The gains move to the right position.
* OnChangeNoSet -- OnChange means that the variable `balance` is only changed when the head unit sends a new balance command.  But this is not useful for our example because the set function is never called.
  * ![](https://lh5.googleusercontent.com/MZQQUrEhkyqGCJ1H-gpCY0Z3YcpYgqMAs8yKAA3xofGnDLRrFHLcSMWaUEyubsldl2o4YRfShz5wjVup1wZyFkTcgVG2z5DVJ_oM_ffOZnWgfexJRZ5V2grIPsPp_2lJ1avFrK42)The gains remain in the center position.
* OnChangeDeferredSet - This behavior means that the ParmaSet only sends a new balance variable when its input changes.  The awe framework calls the set function on a low priority basis. This is the best choice for our balance example.  The balance function does not need critical timing.
  * ![](https://lh3.googleusercontent.com/c99trkiMcgx8M06zr35VTpvblFWY8iF7SFJgefAq_qM284tBsmsUdsG2mwulcE8VqhFh-UbzlZfCw1Ktd60fBZjhK07YjxSt1dGsaY8_gh-83_qDnW4EjPqBK6skfLhyAbkCC4as)The gains move to the right position.
* OnChangeInstantSet - Only change the balance variable when a new command comes from the head unit, but call the set function immediately. This behavior is good to use when it is important to make all changes as fast as you can  to avoid latency.  
  * ![](https://lh3.googleusercontent.com/c99trkiMcgx8M06zr35VTpvblFWY8iF7SFJgefAq_qM284tBsmsUdsG2mwulcE8VqhFh-UbzlZfCw1Ktd60fBZjhK07YjxSt1dGsaY8_gh-83_qDnW4EjPqBK6skfLhyAbkCC4as) The gains are moved to the right position.  

## Inspector

The inspector for this module is displayed by double clicking on the module in AWE\_designer. The setBehavior variable can be changed in real time.

![](../../../.gitbook/assets/8%20%286%29.png)

## Matlab Usage

`M=param_set_module(NAME, DATATYPE, MODVAR)`

Parameter set module. This module can reach into the instance structure of other modules and set parameter values.

### Arguments

* `NAME` - name of the module.
* `DATATYPE` - string specifying the data type of the variable and this is also used for the data type of the input pin. Allowable values are 'float', 'int', or 'fract32'.
* `VARNAME` - specifies the module and variable name using the form:
  * '`MOD.VAR`' where `MOD` is the module name and `VAR` is the variable name. You can also specify internal subsystems using '`SUBSYS.MOD.VAR`'. The ‘\’ can also be used to move up a level.

## Type Definition

```cpp
typedef struct _ModuleParamSet
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 setBehavior;                 // Controls the behavior of the module
   void * modPtr;                     // Points to the module to set
   void * varPtr;                     // Points to the variable to set within the
                                      // module instance structure
} ModuleParamSetClass;
```

## See Also

ParamGet, PresetArrayHandler, SetWireProperties, StatusSet, UpdateSampleRate

