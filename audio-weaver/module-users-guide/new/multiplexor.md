---
description: Selects one of N inputs. No smoothing.
---

# Multiplexor

This module has a variable number of input pins and copies one of these pins to the output pin. Each input pin must have the same number of channels, block size and sample rate. The parameter `index` specifies which of the input pins to route to the output pin. `index` is zero based and must lie in the range of \[0 `numInPin`-1\] where `numInPin` is the number of input pins. If the value of `index` is outside of the allowable range, it is clipped so that it lies in the proper range.

Multiplexor has one output pin, and the properties of the output pin \(sample rate, block size and number of channels\) is the matches the input pins.

This module can handle float, int and fract32 data types. Each input pin must have the same data type.

Note that this module is not smoothed and audible clicks may occur when changing the index at run time. If you need a smoothed clickless operation, use the MultiplexorV2 module instead.

The module has an optional control pin when the argument `indexFromPin` = 1. The module then has `numInPin`+1 input pins and the last one is a control pin of size 1x1 which specifies which input signal to use. The control signal is zero based and in the range \[0 `numInPin`-1\]. The control signal must be an integer with a buffer size of 1 and sample rate of \(system sample rate\)/\(system block size\).

### Arguments

![](../../../.gitbook/assets/0%20%2825%29.png)

* `numInPin` - Determines the number of input pins. The default is 2 input pins.
* `indexFromPin` - If true there is an extra pin added to the bottom of the inputs pins. The default is false. The external `index` input must be of data type integer.

### Variables

![](../../../.gitbook/assets/1%20%2832%29.png)

* `index` - Specifies which input pin to copy to the output pin. When `indexFromPin` = true, `index` can be changed but it has no effect on the output pin. The external `index` takes priority.

## Examples

### Multiplexor with External Index

Two very important points are shown in this example:

*  As the multiplexor switches inputs you can see in the figure below that the transition is not smooth there will be pops and clicks heard.
* This module only switches at the beginning of a block. Thus the sample rate of the Pulse Generator module shown below is 48000/32 = 1500 samples/sec. The block size is one. The wire going from the pulse generator is dashed indicating that it is a control signal.

![](../../../.gitbook/assets/2%20%2825%29.png)

## Inspector

The inspector for the Multiplexor module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to view and change the index value in real time. If the multiplexor has 2 pins to select from, then the inspector will be a checkbox. Unchecked selects pin 0; checked selects pin 1.

![](../../../.gitbook/assets/7%20%288%29%20%281%29.png)

If the multiplexor has more pins, then instead of a checkbox, you will have a droplist:

![](../../../.gitbook/assets/8%20%287%29.png)

## Matlab Usage

`M=multiplexor_module(NAME, NUMINPIN, INDEXFROMPIN)`

Creates a multiplexor module for use in the Audio Weaver environment. The module has multiple input pins and routes 1 of the inputs to the output. Note, this module performs no smoothing and you may get an audible discontinuity when changing the index variable. Use one of the smoothed multiplexor modules to avoid this.

### Arguments

`NAME - name of the module.`

`NUMINPIN - number of input pins (default = 2).`

`INDEXFROMPIN - 1 takes input from INPUT pin (default = 0).`

 `All input pins must have the same block size and number of channels.`

 `M=multiplexor_module(NAME, NUMINPIN, INDEXFROMPIN)`

 `An optional third input argument specifies that the module uses a control`

 `signal. When INDEXFROMPIN = 1, then the module has an additional input`

 `pin (the last one) of size 1x1 (one channel and a block size of 1).`

 `The control signal is integer data and is used based. Values are clipped`

 `to the range [0 NUMINPIN-1].`

## Type Definition

```cpp
typedef struct _ModuleMultiplexor
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 indexPinFlag;                // Specifies index pin available or not.
   INT32 index;                       // Specifies which input pin to route to the
                                      // output. The index is zero based.
} ModuleMultiplexorClass;
```

## See Also

MultiplexorV2, MultiplexorV2Fract32, Router, RouterSmoothed, RouterSmoothedFract32, SampleMultiplexorControl

