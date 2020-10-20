---
description: Smoothly mutes and unmutes
---

# MuteSmoothed MuteNSmoothed

## MuteSmoothed

Smoothly mutes or unmuted audio signals. Muting/unmuting is controlled by the `isMuted` variable. The `isMuted` variable is automatically converted to an internal target gain variable called `gain`:

* `isMuted` is set to false. The `gain` = 1.
* `isMuted` is set to true. The `gain` = 0.

Changes made to the target gain variable are exponentially smoothed \(first order IIR\) at the sample rate, with the time constant determined by the `smoothingTime` variable. The internal variable `currentGain` represents the instantaneous smoothed `gain` that is applied to the data stream. `currentGain` exponentially approaches `gain` with a time constant equal to smoothingTime.

This module supports multichannel data with each channel being simultaneously muted and unmuted by the same gain ramp.

When the module is built, it internally sets currentGain = targetGain and this forces the module to start in a converged state without any startup transient.

#### Variables

![](../../../.gitbook/assets/0%20%2830%29.png)

* `isMuted` - Control variable to mute the output.
* `smoothingTime` - The time constant to determine how fast the output converges to a change in state.

### MuteNSmoothed

MuteNSmoothed is similar to MuteSmoothed, but the MuteNSmooted module is a multichannel smoothly varying mute module. The variable `isMuted` has been replaced by a boolean array `isMuted` which has a separate value for each channel in the signal. The size of `isMuted` is set by the prebuild function. If you don’t see the correct number of channels while in Design mode, you may have to propagate the wire information or correct some wiring problems.

By default, the inspector for the module has the control checkboxes organized in a vertical stack. By setting the argument transpose to true , the control boxes are displayed as a horizontal stack.

#### Arguments

![](../../../.gitbook/assets/1%20%2831%29.png)

* `numChannels` - This is an optional argument. It determines the initial length of the `isMuted` array. In all cases, the array length is updated by pin propagation.
* `transpose` - When true, the inspector is displayed as a horizontal stack.

#### Variables

![](../../../.gitbook/assets/2%20%2834%29.png)

* `smoothingTime` - Time constant used to smooth the output.

#### Array

![](../../../.gitbook/assets/3%20%2820%29.png)

* `isMuted` - A boolean array. The size of the array equals the number of input channels.

## Examples

### Mute vs MuteN

In the signal flow below, six channels of audio are presented to the MuteSmoothed and MuteNSmoothed channels. From the meters note that MuteSmoothed can only perform muting on all channels. While with MuteNSmoothed, you can select which channels are muted.

![](../../../.gitbook/assets/4%20%2821%29.png)

## Inspector

The inspector for both modules can be viewed by double clicking on the module in AWE\_designer. Note that the second inspector is in the transposed position.

![](../../../.gitbook/assets/5%20%289%29.png)

![](../../../.gitbook/assets/6%20%2810%29.png)

## Matlab Usage

 `M=mute_smoothed_module(NAME)`

 Creates a module which smoothly mutes and unmutes.

###  Arguments

*  `NAME` - name of the module.

`M=muten_smoothed_module(NAME, NUMCHANNELS, TRANSPOSE)`

Creates a multichannel mute module in which there is a separate control per channel.

### Arguments

* `NAME` - name of the module.
* `NUMCHANNELS` - optional argument that specifies the initial number of channels in the wire. When the system is built, the number of channels is derived by pin propagation. This value is only for the initial behavior before the system is built. By default, `NUMCHANNELS` = 1.
* `TRANSPOSE` - an optional argument which controls how the inspector for the module is drawn. By default, if there are N channels then the inspector control is a vertical stack of N checkboxes. \(`TRANSPOSE` = 0 by default\). If `TRANSPOSE` = 1, then the inspector is drawn as a horizontal array of N checkboxes.

## Type Definition

```cpp
typedef struct _ModuleMuteSmoothed
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 isMuted;                     // Boolean that controls muting/unmuting.
   FLOAT32 smoothingTime;             // Time constant of the smoothing process
   FLOAT32 currentGain;               // Instantaneous gain applied by the module.
                                      // This is also the starting gain of the
                                      // module.
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
   FLOAT32 gain;                      // Target gain.
} ModuleMuteSmoothedClass;

typedef struct _ModuleMuteNSmoothed
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure
   FLOAT32 smoothingTime;             // Time constant of the smoothing process
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
   INT32* isMuted;                    // Boolean that controls muting/unmuting. One
                                      // per channel.
   FLOAT32* currentGain;              // Instantaneous gain applied by the module.
                                      // This is also the starting gain of the
                                      // module.
   FLOAT32* gain;                     // Target gain.
} ModuleMuteNSmoothedClass;
```

## See Also

Ducker DuckerFract32 MuteLinearFade MuteNSmoothed MuteNSmoothedFract32 MuteSmoothedFract32 MuteSync MuteSyncFract32 MuteUnmute MuteUnmuteFract32 SoloMute SoloMuteFract32 VolumeControl

