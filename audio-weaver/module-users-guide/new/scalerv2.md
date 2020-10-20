---
description: General purpose scaler with a single gain
---

# ScalerV2

General purpose scaler \(or gain\) module for floating-point signals. This module has a single parameter "gain" which specifies the gain to apply. The same gain is applied to all channels.

Gain is interpreted as either a linear value or as a dB value based on the isDB module variable. By default, isDB=true and the gain parameter is treated as a dB value. If you set isDB=false then the gain parameter is treated as a linear value.

The module can also switch between smoothed and unsmoothed gain changes. You enable smoothing by setting the parameter `smoothingTime` to a non-zero value. If `smoothingTime`=0 then no smoothing is performed. The module is slightly more efficient when no smoothing occurs.

The module supports multiple input and output pins using the argument `numPins`. By default, `numPins`=1 and the module has a single input and output pin. If you set NUMPINS &gt; 1 then you can scale multiple signals. The same gain variable is used across signals.

By default the module has no control pin and the gain is taken from the instance variable `gain`. The argument `isControl` controls whether the module has an input control pin. By default `isControl` is false and the module is not controllable. If you set `isControl` to true then the module will have one extra input pin. In this case, the gain is taken from the first input pin. The control pin accepts only mono wires and the gain is taken from the first sample in the buffer. The total gain applied is then a combination of the value on the control pin and the gain parameter. For dB gains, the total gain applied is controlPinValue + gain. For linear gains, the total gain applied is controlPinValue \* gain.

The description above applies to real data. The module can also handle complex data but in the case of complex data no smoothing is performed.

### Arguments

![](../../../.gitbook/assets/0%20%2826%29.png)

* `isControl` - Determines if the first input pin is a control input.
  * `false` - no control pin
  * `true` - the first input pin is a control input.

### Variables

![](../../../.gitbook/assets/1%20%2827%29.png)

* `gain` - The gain applied to all input channels. Can be either dB or linear.
* `smoothingTime` - The time constant for the smoothing process.
* `isDB` - Determines if gain is linear or dB.
  * `false` - The gain is linear.
  * `true` - The gain is dB.

Note that `gain` and `Min` and `Max` range are not changed when `isDB` changes.

## Examples

### No Control

In the figure below we show the simple case of adding 6 dB of gain to a signal. The input to the scaler module measures -20 dB. The output of the scaler module measures -14 dB. Since `isDB` = true, to predict the output energy we simply add. -20 dB + 6 dB = -14 dB.

![](../../../.gitbook/assets/2%20%2827%29.png)

### With Control

This example is similar to the one above, except that a control input pin is added to the scaler module by setting the `isControl` variable to true. The gain input is a value of 3 dB. As we look at the output meter we see that the reading is -11 dB. This is because the input control gain is combined with the internal `gain` variable which is 6 dB. Therefore the output dB is -20 dB + 3 dB + 6 dB = 11 dB.

![](../../../.gitbook/assets/3%20%2821%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module. The gain variable can be modified in real time. But the inspector can only be viewed when `isControl` = false.

![](../../../.gitbook/assets/4%20%2815%29.png)

## Matlab Usage

`M = scaler_v2_module(NAME, ISCONTROL, NUMPINS)`

General purpose scaler module with a single gain applied across all channels. The module can be configured for linear or dB operation as well as smoothed and unsmoothed gain changes.

### Arguments

* `NAME` - name of the module.
* `ISCONTROL` - Boolean. Does the module have a control pin for the gain. Default = 0 and the gain to apply is taken from the inspector.
* `NUMPINS` - number of input pins. Default = 1.

## Type Definition

```cpp
typedef struct _ModuleScalerV2
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   FLOAT32 gain;                      // Gain in either linear or dB units.
   FLOAT32 smoothingTime;             // Time constant of the smoothing process (0 =
                                      // unsmoothed).
   INT32 isDB;                        // Selects between linear (=0) and dB (=1)
                                      // operation.
   FLOAT32 targetGain;                // Target gain in linear units.
   FLOAT32 currentGain;               // Instantaneous gain applied by the module.
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
} ModuleScalerV2Class;
```

## See Also

ScaleOffset, ScaleOffsetFract32, ScaleOffsetInt32, ScalerControl, ScalerControlBlockSmoothedFract32, ScalerControlSmoothedFract32, ScalerDBControl, ScalerDBFract32, ScalerDBSmoothedFract32, ScalerFract32, ScalerNControl, ScalerNDBSmoothedFract32, ScalerNFract32, ScalerNSmoothedFract32, ScalerNV2, ScalerSmoothedFract32, ScalerV2

