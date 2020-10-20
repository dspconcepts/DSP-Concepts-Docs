---
description: General purpose scaler with separate gains per channel.
---

# ScalerNV2

The ScalerNV2 is a general purpose scaler \(or gain\) module for floating-point signals. This module has a single gain \(`masterGain`\) which is applied to all channels plus a separate per channel gain \(`trimGain`\). For each channel, `masterGain` and `trimGain`\[n\] are combined to form the target gain value.

The gains are interpreted as either linear or dB values based on the `isDB` module variable. By default, `isDB`=1 and the gains are treated as dB values. If you set `isDB`=0 then the gains are treated as linear values. When in dB mode, `targetGain = masterGain + trimGain`\[n\]. When in linear mode, `targetGain = masterGain * trimGain`\[n\].

The module can also switch between smoothed and unsmoothed gain changes. You enable smoothing by setting the parameter `smoothingTime` to a non-zero value. If `smoothingTime`=0 then no smoothing is performed. The module is slightly more efficient when no smoothing occurs.

The module argument `numChannels` sets the initial number of channels. This allows you to configure the module for the proper number of channels before the system is built. After the system is built, `numChannels` is no longer used and the number of channels is determined from the input wire.

### Arguments

![](../../../.gitbook/assets/0%20%2834%29.png)

* `numChannels` - Initial number of channels that are used before the system is built. After that the number of channels is determined by pin propagation.

### Variables

![](../../../.gitbook/assets/1%20%2830%29.png)

* `masterGain` - The overall gain.
* `smoothingTime` - The time constant used in the smoothing process when the gains are changed.
* `isDB` - Selects between linear and dB. It should be noted that masterGain and trimGains are not changed when this variable changes. For example a 0 dB gain will now be a 0 linear gain and may not be what you want!

### Array

![](../../../.gitbook/assets/2%20%2829%29.png)

* `trimGain` - Trim gains applied to each individual channel.

## Examples

### Surround 5.1 Mix Down

In the signal flow below, away is shown to convert Surround 5.1 to Stereo. Surround 5.1 format has 6 channels: lf,rf,cf,lfe,ls,rs.

![](../../../.gitbook/assets/3%20%2824%29.png)

The lfe channel is not needed since it is usually a mono signal. It is removed using Router1. The equation to calculate the left stereo channel is:

$$
Lo=lf+ls+cf/√2
$$

The equation to calculate the right stereo channel is:

$$
Ro=rf+rs+cf/√2
$$

ScalerN1 is configured for linear gains and the gain settings are as shown below:

![](../../../.gitbook/assets/4%20%2816%29.png)

The trim gains for ScalerN2 are:

![](../../../.gitbook/assets/5%20%2810%29.png)

The adders finish the linear combination and the stereo signal is complete.

## Inspector

The inspector for this module can be viewed by double clicking on the module. An example of the inspector is shown above. The `masterGain` and `trimGains` can be changed in real time.

## Matlab Usage

`M = scalern_v2_module(NAME, NUMCHANNELS)`

General purpose scaler module with a separate gain per channel. The module can be configured for linear or dB operation as well as smoothed and unsmoothed gain changes.

### Arguments

* `NAME` - name of the module.
* `NUMCHANNELS` - default number of channels. Default = 1. This is used to draw the initial inspector. The actual number of channels processed by the module is determined by pin propagation.

## Type Definition

```cpp
typedef struct _ModuleScalerNV2
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   FLOAT32 masterGain;                // Overall gain to apply.
   FLOAT32 smoothingTime;             // Time constant of the smoothing process (0 =
                                      // unsmoothed).
   INT32 isDB;                        // Selects between linear (=0) and dB (=1)
                                      // operation
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
   FLOAT32* trimGain;                 // Array of trim gains, one per channel
   FLOAT32* targetGain;               // Computed target gains in linear units
   FLOAT32* currentGain;              // Instantaneous gains. These ramp towards
                                      // targetGain
} ModuleScalerNV2Class;
```

## See Also

ScaleOffset, ScaleOffsetFract32, ScaleOffsetInt3,2 ScalerControl, ScalerControlBlockSmoothedFract32, ScalerControlSmoothedFract32, ScalerDBControl, ScalerDBFract32, ScalerDBSmoothedFract32, ScalerFract32, ScalerNControl, ScalerNDBSmoothedFract32, ScalerNFract32, ScalerNSmoothedFract32, ScalerNV2, ScalerSmoothedFract32, ScalerV2

