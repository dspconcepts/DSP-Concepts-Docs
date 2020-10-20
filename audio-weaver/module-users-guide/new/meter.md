---
description: Peak and RMS meter module
---

# Meter

This module provides a flexible level meter that can operate in several modes. The module has a single multichannel input and separately displays a meter for each channel. The meter can be configured to conform to IEC 60280-10 \(peak meters\) and IEC 60280-17 \(VU Meter\) specifications.

### Variables

![](../../../.gitbook/assets/0%20%2832%29.png)

The meterType parameter is used to configure the module as follows:

* FastPeakSample \(Type=0\) - Meter implements a peak meter according to IEC 60280-10 in "fast mode". The attackTime is fixed at 5 msec and the release time is fixed to 1087 msec.
* SlowPeakSample \(Type=1\) - Meter implements a peak meter according to IEC 60280-10 in "slow mode". The attackTime is fixed at 10 msec and the release time is fixed to 1450 msec.
* VUMeterSample \(Type=2\) - Meter implements a meter according to IEC 60280-17. This corresponds to the industry standard "VU meter" definition. The attack and release times are 65 msec.
* CustomSample \(Type=3\) - Meter implements a custom peak meter. You have to manually set the attackTime and releaseTime. Currently the attack and release time can only be set via the Matlab API.
* InstantSample \(Type=4\) - Meter attack and release time are instantaneous. Attack and release times = 0 msec. The meter value equals the last value in each block.
* FastPeakBlock \(Type=16\) - Meter implements a computationally cheap approximation to IEC 60280-10 in "fast mode". The attackTime is fixed at 5 msec and the release time is fixed to 1087 msec.
* SlowPeakBlock \(Type=17\) - Meter implements a computationally cheap approximation to IEC 60280-10 in "slow mode". The attackTime is fixed at 10 msec and the release time is fixed to 1450 msec.
* VUMeterBlock \(Type=18\) - Meter implements a computationally cheap approximation to IEC 60280-17. This corresponds to the industry standard "VU meter" definition. The attack and release times are 65 msec.
* CustomBlock \(Type=19\) - Meter implements a custom peak meter in a computationally fast manner. You have to manually set the attackTime and releaseTime. Currently the attack and release time can only be set via the Matlab API.
* InstantBlock \(Type=20\) - Meter implements an instantaneous peak meter. The attack and decay times are zero. Basically, the max abs value of each block is displayed for each block without time constants.

In meterTypes 0 to 4, the peak attack and release are performed on a sample-by-sample basis in the envelope follower. In meterTypes 16 to 20, the peak absolute value for the entire input block is found and this one value is passed through the envelope follower.

## Examples

### Fast Peak Sample Algorithm

The Meter module is used in most signal flows. It is a very useful tool. For example, when debugging a signal flow one of the first things you need to know is if you have inputs. The Meter module inspector can easily show you this. For this example let’s dive down and see what is happening inside this module.

In the signal flow below, we have a sine wave with a peak to peak amplitude of 0.5 \(that is, +/- 0.25\). Meter2 shows a reading of -12.17 dB in the Meter inspector shown below. There is a lower signal chain consisting of MaxAbs, AttackRelease, and Db20 modules. The results of this chain are shown in the Sink1 inspector below. As you can see the results are very close. This is because the time constants used in the AttackRelease module are the same as those used in the Meter2 module.

![](../../../.gitbook/assets/1%20%2828%29.png)

![](../../../.gitbook/assets/2%20%2828%29.png)

## Inspector

The inspector for the Meter module is shown below. It can be viewed by double clicking on the Meter module in AWE\_designer. The inspector allows you to view DB output levels in real time.

![](../../../.gitbook/assets/3%20%2826%29.png)

If the meter has 2 input channels, then the inspector will show a bar graph per channel:

![](../../../.gitbook/assets/4%20%2819%29.png)

And finally, if the input to the meter module is all zeros, it will be shown as:

![](../../../.gitbook/assets/5%20%2811%29.png)

## Matlab Usage

`M=meter_module(NAME)`

Creates a meter module for use in the Audio Weaver environment. The module has a single multichannel input pin.

#### Arguments:

* `NAME` - name of the module.

## Type Definition

```cpp
typedef struct _ModuleMeter
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 meterType;                   // Operating mode of the meter. Selects
                                      // between peak and RMS calculations. See the
                                      // Description section for more details.
   FLOAT32 attackTime;                // Attack time of the meter. Specifies how
                                      // quickly the meter value rises.
   FLOAT32 releaseTime;               // Release time of the meter. Specifies how
                                      // quickly the meter decays.
   FLOAT32 attackCoeff;               // Internal coefficient that realizes the
                                      // attack time.
   FLOAT32 releaseCoeff;              // Internal coefficient that realizes the
                                      // release time. 
   FLOAT32* value;                    // Array of meter output values, one per
                                      // channel.
} ModuleMeterClass;
```

## See Also

MeterFract32, Sink

