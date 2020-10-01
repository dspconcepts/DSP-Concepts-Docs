---
description: >-
  Deinterleaves a multichannel audio signal into separate mono signals.
  Interleaves multiple audio signals.
---

# Deinterleave, Interleave

## Deinterleave

Deinterleaves a multichannel input signal into separate mono output channels. This is useful when you need to operate separately on individual channels.

The module has a single multichannel input pin and multiple mono output pins. At construction time, you must specify the number of output pins, which must equal the number of channels in the input. The blockSize, sample rate and data type of the input pin is propagated to all output pins.

The module can also operate on real or complex data.

### Arguments

![](../../../.gitbook/assets/0%20%2814%29.png)

* `numOut` - The number of output pins. This is required and cannot be implied by the number of input channels.

## Interleave

The interleave module takes N audio signals and produces a single output, where the N input signals are interleaved together. This module allows for different signal channels to be bundled into a single wire, which is useful when creating uncongested layouts. Each input pin can have an arbitrary number of interleaved channels, but they must all be the same data type, blockSize, and sampleRate. The output contains all of the channels on the first input pin followed by all of the channels of the second input pin, and so on, for all input pins. There are no restrictions on the number of channels on each pin.

### Arguments

![](../../../.gitbook/assets/1%20%2820%29.png)

* `numIn` - The number of input pins on the module. Each input pin can have a different number of channels. Each channel must be of the same data type, block size and sample rate.

## Examples

### Multi-channel Interleave

In the figure below, there is an Interleave module paired with a Deinterleave module. The module SourceFloating123 has three floating point output channels. The module Source45 has two floating point output channels. These are acceptable inputs for the Interleave module because the number of channels in each input pin can be different. As you can see in the Interleave Display below, the channels in the output pin are interleaved correctly. On the other hand the Deinterleave module has one channel for each output pin. You must tell the Deinterleave module how many input channels \(= number of output pins\) are required before run time. It cannot be implied. Another very important point should be mentioned here. While Deinterleave and Interleave can work with any 32 bit data type, these data types cannot be mixed on the input pins of Interleave.

![](../../../.gitbook/assets/2%20%2822%29.png)

## Inspector

Deinterleave and Interleave modules do not have inspectors.

## Matlab Usage

`M=deinterleave_module(NAME, NUMOUT)`

Creates a deinterleave module for use in the Audio Weaver environment.

Arguments:

* `NAME` - name of the module.
* `NUMOUT` - number of output pins.

`M=interleave_module(NAME, NUMIN)`

Creates a multi-input interleave module for use in the Audio Weaver environment.

Arguments:

 `NAME` - name of the module.

 `NUMIN` - number of input pins.

## Type Definition

```cpp
typedef struct _ModuleDeinterleave
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 //structure
} ModuleDeinterleaveClass;
typedef struct _ModuleInterleave
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 // structure
} ModuleInterleaveClass;
```

## See Also

Router, Multiplexor, MultiplexorV2

