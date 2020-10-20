---
description: Smooth scaler in which the gain (in linear units) is taken from an input pin
---

# ScalerControl

The ScalerControl module smoothly scales multi-channel input data by a single gain value. The first pin is the control pin and can have only one channel. The `gain` \(in linear units\) value is provided as the first sample of the first pin. Meaning when the buffer size of input pin 1 is greater than 1, the first sample in the input buffer is used as the `gain`. The module has N+1 input pins and N number of output pins. The module includes smoothing and internally, `currentGain` represents the instantaneous smoothed gain that is applied. `currentGain` exponentially approaches gain with a time constant equal to `smoothingTime`.

### Arguments

![](../../../.gitbook/assets/0%20%2833%29.png)

* `numPins` - The number of input and output data pins. This does not include the `gain` pin.

### Variables

![](../../../.gitbook/assets/1%20%2825%29.png)

* `smoothingTime` - The time constant used in the smoothing filter.

## Examples

### Scaler Control with Smoothing

The figure below shows a ScalerControl module with a `smoothingTime` of 1 msec. Input pin1 is a square wave -1 to 1 peak to peak, this is our gain signal. Input pin2 is a DC constant of 1.0. The output pin is plotted in the Sink Display inspector. As you can see, because of the smoothing filter the output exponentially approaches -1.0 and 1.0.

![](../../../.gitbook/assets/2%20%2824%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module in Audio Weaver. The `smoothingTime` variable can be changed realtime.

![](../../../.gitbook/assets/3%20%2823%29.png)

## Matlab Usage

`M=scaler_control_module(NAME,NUMPINS)`

Creates a smoothly varying scaler module which has a variable number of input and output pins and each pin can be multichannel and will have the same gain applied to it. The first sample of the first pin is used as a target gain for the block. The first pin is the control pin and can only have one channel. The module can also be used to scale complex data. However, the smoothing coefficient is ignored and no smoothing is performed in this case.

### Arguments

* `NAME` - name of the module.
* `NUMPINS` - number of audio input/output pins. By default, `NUMPINS`=1.

## Type Definition

```cpp
typedef struct _ModuleScalerControl
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   FLOAT32 smoothingTime;             // Time constant of the smoothing process
   FLOAT32 currentGain;               // Instantaneous gain applied by the module.
   FLOAT32 smoothingCoeff;            // Smoothing coefficient
} ModuleScalerControlClass;
```

## See Also

ScaleOffset, ScaleOffsetFract32, ScaleOffsetInt32, ScalerControl ScalerControlBlockSmoothedFract32, ScalerControlSmoothedFract32, ScalerDBControl, ScalerDBFract32, ScalerDBSmoothedFract32, ScalerFract32 ScalerNControl, ScalerNDBSmoothedFract32, ScalerNFract32, ScalerNSmoothedFract32, ScalerNV2, ScalerSmoothedFract32, ScalerV2

