---
description: Scales a signal and then adds an offset
---

# ScaleOffset

The ScaleOffset module scales the input signal, and adds a fixed \(DC\) offset. The module has 1 input pin and it supports multichannel processing. The module is useful for specific nonlinearities, as well as in dynamics processing, where the scale and offset can occur in the log domain.

The output is computed on a sample by sample basis according to the formula:

$$
y[n] = gain * x[n] + offset
$$

It is often easier to use the ScaleOffset module rather than a separate DCSourceV2 module followed by an Adder module.

### Variables

![](../../../.gitbook/assets/0%20%2828%29.png)

* `gain` - The linear floating point gain. There is no DB option, the gain is always linear.
* `Offset` - DC floating point offset.

## Examples

### Scaled and Offset Sine Wave

In the signal flow below, a sine wave is passed through a ScaleOffset module and displayed in the SinkDisplay inspector. The original signal has a max value of 1 and a min value of -1. The `gain` for the ScaleOffset module is 0.5 and the `offset` is 0.5. The output of the ScaleOffset module is now:

![](../../../.gitbook/assets/1%20%2833%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module in Audio Weaver. The `gain` and `offset` variables can be changed in real time.

![](../../../.gitbook/assets/2%20%2831%29.png)

## Matlab Usage

`M=scale_offset_module(NAME)`

Creates a scale and offset object for use with the Audio Weaver environment.

###  Arguments

*  `NAME` - name of the module.

## Type Definition

```cpp
typedef struct _ModuleScaleOffset
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   FLOAT32 gain;                      // Linear gain.
   FLOAT32 offset;                    // DC offset.
} ModuleScaleOffsetClass;
```

## See Also

ScaleOffsetFract32, ScaleOffsetInt32

