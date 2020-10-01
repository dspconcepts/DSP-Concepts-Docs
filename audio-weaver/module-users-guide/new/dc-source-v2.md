---
description: Outputs a constant DC value with optional input control pin to scale DC value.
---

# DC Source V2

The DC source module has an internal variable holding one data value. The module continuously fills the output wire with the internal data value. This module also has an optional input control pin to scale the internal data value. This control pin can be multi-channel. The output blockSize and sample rate can be specified. If not specified, the values are taken from the system input.

### Arguments

![](../../../.gitbook/assets/0%20%2824%29.png)

* `numChannels` - The number of output channels. The same data appears on all channels.
* `blockSize` - Block size of the data generated. If left blank the block size of the first input pin of the containing system will be used.
* `sampleRate` - Sample rate of data generated. If left blank the sample rate of the first input pin of the containing system will be used.
* `controlPin` - A boolean to add a control input pin. Default is false.

### Variables

![](../../../.gitbook/assets/1%20%2817%29.png)

* `value` - All data generated will have this value. All channels will have the same value.

## Examples

### Separate Values for Each Channel

Let’s say that we need two output channels and the values on each channel needs to be different. We can do this by setting the `value` variable to 1. Also set the `controlPin` argument to true and the `numChannels` argument to 2. Now we can input a signal with two channels on the control pin which will scale `value`. The figure below shows the results, output channel 1 is a constant 0.25 and channel 2 is a constant 0.7071.

![](../../../.gitbook/assets/2%20%2815%29.png)

## Inspector

The inspector for the DC Source V2 module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to change the `value` variable in real time.

![](../../../.gitbook/assets/3%20%2813%29.png)

## Matlab Usage

`M=dc_source_v2_module(NAME, NUMCHANNELS, BLOCKSIZE, SAMPLERATE, CONTROLPIN)`

Creates a source module that allows you to inject DC test data into

 the audio processing layout. Also has optional input control pin to scale

 the data value to be written on the output pin.

 Arguments:

* `NAME` - name of the module.
* `NUMCHANNELS` - number of interleaved channels in each output pin.
* `BLOCKSIZE` - number of samples per output channel.
* `SAMPLERATE` - sample rate of the output signal, in Hz.
* `CONTROLPIN` - boolean to add control input pin or not.

## Type Definition

```cpp
typedef struct _ModuleDCSourceV2
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 // structure
 FLOAT32 value; // Data Value
} ModuleDCSourceV2Class;
```

## See Also

BooleanSource, DCSourceInt, DCSourceV2Fract32, ImpulseMsecSource, ImpulseMsecSourceFract32, ImpulseSourceFract32, PeriodicSource, PeriodicSourceFract32, Source, SourceFract32, SourceInt, WAVInterp16OneShotSourceFract32, ZeroSource, Sink

