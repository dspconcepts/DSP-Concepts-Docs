---
description: Calculates gain used to realize soft-knee peak limiters.
---

# AGC Limiter Core

## Overview

The limiter core is a basic building block for creating Automatic Gain Control \(AGC\) systems. The function of a limiter is to prevent the output of an audio system from clipping. The limiter core provides time varying gain that can be used to limit the output level of an audio signal. It is designed to take as its input the output of the MaxAbs module. The MaxAbs module can accept multi-channel inputs but only outputs the maximum of the absolute value taken across all channels. The AGCLimiterCore module outputs a time varying gain on the mono output pin which is used to limit the output level of an audio system. The time varying gain output is calculated at the sample rate, which aids in reducing the gain of fast transients.

The module stores the gain computed for the last sample in the block to the currentGain state variable. This variable is shown on the inspector and reflects the output gain computed by this module.

### **Variables**

![](../../../.gitbook/assets/0%20%2820%29.png)

The variables are graphically depicted in the figure below. The static curve plots the output level versus the input level. Where the curve has a slope of 1 would indicate that the time varying gain output equals 0 dB.

Where:

* `threshold` - When the input level is above the threshold the output gain is reduced.
* `gain` - Value used to scale the output of the AGC Core. The gain is uniformly applied and is equivalent to having a scaler following the AGCLimiterCore. The gain is also applied for signal levels below the threshold.
* `kneeDepth` - Softens the transition as input level transitions between no limiting and limiting. The knee provides a gentle polynomial interpolation between the threshold and the requested slope. The kneeDepth extends +/- around the threshold. That is, the knee starts at threshold-kneeDepth and extends to threshold+kneeDepth.
* `ratio` - Slope of the output attenuation when the input level is above threshold. slope = 1.0 - \(1.0/`ratio`\). A high ratio provides hard limiting, close to 1 provides gentle limiting. If the ratio equals 10, then the input signal has to increase by 10 dB above the threshold for the output to increase by 1 dB.
* `attackTime` - The speed at which a limiter responds to an increase in sound level.
* `decayTime` - The speed at which the limiter’s effect is relaxed after the sound level drops back under its threshold.

![](../../../.gitbook/assets/1%20%2819%29.png)

## Examples

### Mono Channel Limiter

In the signal flow below a sine wave is passed through Scaler1 and then presented to a basic one channel limiter. The Scaler1 output is also passed to a clipping module that clips the sine at 1 and -1. The output of the limiter and ClipAsym1 are then displayed in a Sink Display module for comparison.

The basic limiter consists of the modules `MaxAbs1`, `LimiterCore1`, `DelayMsec1` and `Mult1`. The delay module provides “look ahead” and gives the AGCLimiterCore some time to respond to sudden changes in level.

![](../../../.gitbook/assets/2%20%2821%29.png)

For the plot below the Scaler1 gain was set to 3 dB. Since the sine wave’s amplitude ranges from 1 to -1 the input to the limiter is now greater than 0 dBFS. The blue plot shows the clipped sine wave. The black plot shows the output of the limiter. The signal level has been reduced to keep the signal below the threshold of -3 dB.

![](../../../.gitbook/assets/3%20%2815%29.png)

For the plot below the Scaler1 gain was set to 0 dB. Notice that now the blue plot is no longer clipping and has a range from 1 to -1. So why is the range of the output of the limiter smaller? You will notice that the threshold of the LimiteCore1 module is set to -3 dB. This means that we have 3 dB of headroom. The output level will be reduced by for any inputs above -3 dBFS.

![](../../../.gitbook/assets/4%20%289%29.png)

In the final plot below Scaler1 gain has been set to -3 dB. The blue and black plot are now equal because the limiter input level is equal to the threshold.

![](../../../.gitbook/assets/5%20%286%29.png)

### Stereo Limiter

This next example, shown in the figure below, is similar to the first but now there is a stereo input to the limiter. This is possible because the `MaxAbs` module selects the maximum absolute value for all input channels. The `LimiterCore1` module still only outputs one time varying gain. This is important when you want to keep the ratio of output level constant on a stereo channel. The stereo image remains stationary as the gain changes.

![](../../../.gitbook/assets/6%20%288%29.png)

### Crossover With Limiters

All of the examples thus far are single band and operate on the entire signal at each stage. This can lead to pumping in which a loud low frequency signal \(like a drum beat\) can cause the entire signal to reduce in level. To avoid this, it is better to treat the low and high frequencies separately. This brings us to a multiband approach in which the signal is split into two components using a crossover and processes the high and low frequencies separately.

The signal flow below shows an implementation of a dual band approach. Notice that the limiter core for the lower band uses slower time constants to handle the higher energy in the lower band. Now the gains for each limiter will be different. When the two bands are added back together clipping can occur. This is shown with the clipping indicators. To prevent clipping a soft clip module is added.

![](../../../.gitbook/assets/7%20%286%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to adjust the `threshold`, `gain` , `ratio`, `knee depth`, `attackTime` and `decayTime` variables in real time.

![](../../../.gitbook/assets/8%20%285%29.png)

## Matlab Usage

To create an `AGCLimiterCore` module using Matlab scripts the function `agc_limiter_core_module` is called.

 `M=agc_limiter_core_module(NAME)`

### Arguments

> >

`NAME` - name of the module.

## Type Definition

```cpp
typedef struct _ModuleAGCLimiterCore
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance structure
    FLOAT32 threshold; // Amplitude level at which the AGC Limiter Core
                       // reduces its output gain value
    FLOAT32 gain; // Value used to scale the output of the AGC Limiter Core
    FLOAT32 slope; // Internal derived variable which holds the slope of the compression curve
    FLOAT32 kneeDepth; // Knee depth controls the sharpness of the
                       // transition between no limiting and limiting
    FLOAT32 ratio; // Slope of the output attenuation when the
    // signal is above threshold - derived from the
    // standard compression ratio parameter by the
    // formula slope = 1.0 - (1.0/ratio)
    FLOAT32 attackTime; // Envelope detector attack time constant
    FLOAT32 decayTime; // Envelope detector decay time constant
    FLOAT32 currentGain; // Instantaneous gain computed by the block
    FLOAT32 sharpnessFactor; // Internal derived variable which is used to implement the soft knee
    FLOAT32 attackCoeff; // Internal derived variable which implements the attackTime
    FLOAT32 decayCoeff; // Internal derived variable which implements the decayTime
    FLOAT32 envState; // Holds the instantaneous state of the envelope detector
} ModuleAGCLimiterCoreClass;
```

## See Also

AGCAttackRelease AGCAttackReleaseFract32 AGCCompressorCore AGCCompressorCoreFract32 AGCCore AGCCoreFract32 AGCLimiterCoreFract32 AGCMultiplierFract32 AGCNoiseGateCore AGCVariableAttackRelease

