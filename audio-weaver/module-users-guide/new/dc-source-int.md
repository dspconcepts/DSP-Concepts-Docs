---
description: Outputs a constant 32-bit integer value.
---

# DC Source Int

The DC source integer module has an internal variable holding one data value. The module continuously fills the output wire with the internal data value. The data value is a 32-bit signed integer with a range of \[-2,147,483,647 +2,147,283,648\]. When the number of channels is greater than 1, each channel has the same constant value.

### Arguments

![](../../../.gitbook/assets/0%20%2816%29.png)

* `numChannels` - Number of output channels, cannot be implied. Defaults to 1.
* `blockSize` - Number of samples to be output on each channel. By default this is empty and the module will use the block size of the first input pin of containing system..
* `sampleRate` - Sample rate of data. By default this is empty and the module will use the sample rate of the first input pin of the containing system.

### Variables

![](../../../.gitbook/assets/1%20%2815%29.png)

* `value` - Constant 32 bit integer value that is output on all channels. The range can be changed as long as it fits within -2,147,483,647 &lt;= value &lt;= 2,147,283,648. If a floating point number is entered, it is rounded to the nearest integer.

## Examples

In the signal flow below the outputs of two DC Source Int modules are plotted. You will notice that because the DC Source Int module outputs integer 2 data values that a Sink Int module is required to plot the data. You may wonder why we do not need an integer Interleave module. This is because the interleave and deinterleave modules are only concerned with the size of the data word and not the data type. Since all data types in Audio Weaver are 32 bits, all data types will work with Interleave and Deinterleave.

![](../../../.gitbook/assets/2%20%2816%29.png)

## Inspector

The inspector for the DC Source Int module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to change the `value` variable in real time.

![](../../../.gitbook/assets/3%20%2814%29.png)

## Matlab Usage

`M=dc_source_int_module(NAME, NUMCHANNELS, BLOCKSIZE, SAMPLERATE)`

Creates a source module that allows you to inject DC test data into the audio processing layout.

Arguments:

* `NAME` - name of the module.
* `NUMCHANNELS` - number of interleaved channels in each output pin.
* `BLOCKSIZE` - number of samples per output channel.
* `SAMPLERATE` - sample rate of the output signal, in Hz.

## Type Definition

```cpp
typedef struct _ModuleDCSourceInt
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                    // structure
 INT32 value;                       // Data Value
} ModuleDCSourceIntClass;
```

## See Also

BooleanSource, DCSourceV2, DCSourceV2Fract32, ImpulseMsecSource, ImpulseMsecSourceFract32, ImpulseSourceFract32, PeriodicSource, PeriodicSourceFract32, Source, SourceFract32, SourceInt, WAVInterp16OneShotSourceFract32, ZeroSource Sink

