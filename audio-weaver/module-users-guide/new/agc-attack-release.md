---
description: Envelope follower with programmable attack and release times.
---

# AGC Attack Release

AGC Attack Release is one of the basic building blocks for creating dynamics processors such as compressors and limiters. The module accepts an N-channel buffer of amplitude samples and produces an output buffer the same size as the input. An envelope detector is applied to each input channel with configurable attack and release times. The attack time applies when the current input value is larger than the output of the envelope detector. Similarly, the release time applies when the current input value is smaller than the output of the envelope detector. To learn more on how this module can be applied to audio applications see the “Audio Weaver User’s Guide”.

**Arguments**

![](../../../.gitbook/assets/0%20%2812%29.png)

`nchan` - Number of input channels. Each channel is processed separately. Note that this is the initial number of channels. Once the module is wired up, it will inherent the number of channels from the input wire.

**Variables**

![](../../../.gitbook/assets/1%20%2812%29.png)

`attackTime` - The time constant used when the envelope detector input is greater than the output. In milliseconds.

`releaseTime` - The time constant used when the envelope detector input is less than the output. In milliseconds.

## Example

### Envelope Detect

In this example a periodic pulse is created using the PulseGen module. The pulse has a period of 1.8 msec and on time of 0.5 msec. \(It is on for 0.5 msec and off for 1.3 msec.\) When the amplitude of the pulse is greater than the output of the Attack Release module, the algorithm enters the attack mode and uses the 0.1 msec time constant. When the amplitude of the pulse is less than the output of the Attack Release module, the algorithm enters the release mode and uses the 1 msec time constant. Since the attack time is smaller than the release time, the output signal level increases quickly \(it attacks quickly\) and then decays more slowly \(the release time\).

![](../../../.gitbook/assets/2%20%2812%29.png)

### Noise Floor Tracker

Another useful application of this module is to be able to track the noise floor of a signal. In this case, you set a very slow attack time and a very fast release time. With this tuning, the module output tracks the smallest signal values and follows the noise floor of the signal.

![](../../../.gitbook/assets/3%20%289%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to adjust the attackTime and releaseTime variables in real time.

![](../../../.gitbook/assets/4%20%288%29.png)

## Matlab Usage

To create an AGCAttackRelease module using Matlab scripts the function agc\_attack\_release\_module.m is called.

The function header is:

 `M=agc_attack_release_module(NAME, NCHAN)`

Arguments:

`NAME` - name of the module.

`NCHAN` - initial number of channels.

## Type Definition

```cpp
typedef struct _ModuleAGCAttackRelease
{
ModuleInstanceDescriptor instance; // Common AWE module instance structure
FLOAT32 attackTime; // attack time constant.
FLOAT32 releaseTime; // release time constant.
 FLOAT32 attackCoeff; // Calculated filter coefficient for attack time.
 FLOAT32 releaseCoeff; // Calculated filter coefficient for release time.
 FLOAT32* envStates; // State vectors for internal filters.
} ModuleAGCAttackReleaseClass;
```

## See Also

AGCAttackReleaseFract32, AGCCompressorCore, AGCCompressorCoreFract32, AGCCore, AGCCoreFract32, AGCLimiterCore, AGCLimiterCoreFract32, AGCMultiplierFract32, AGCNoiseGateCore, AGCVariableAttackRelease

