---
description: Pink noise generator
---

# PinkNoise

This subsystem generates pink noise by combining a white noise generater and a filter The RMS output level is 0 dB and peak values are well above 0 dB. Internally the module generates white noise and filters it by a PinkFilter module.

### Arguments

![](../../../.gitbook/assets/0%20%2829%29.png)

* `SR` - Sample rate, if left blank it defaults to the sample rate of the first system input pin.
* `numChannels` - The number of statistically independent output channels.
* `blockSize` - Block size of the output channels. If left blank it defaults to the block size of the first system input pin.

## Examples

### Pink Noise vs White Noise

The figures below show a simple example of calculating the spectrum of pink noise and white noise filtered by the module PinkFilter. As you can see in the plots below, spectrally they are the same.

![](../../../.gitbook/assets/1%20%2824%29.png)

![](../../../.gitbook/assets/2%20%2823%29.png)

## Inspector

This module does not have an inspector.

## Matlab Usage

`SYS=pink_noise_subsystem(NAME, SR, NUMCHANNELS, BLOCKSIZE)`

Creates a module which outputs pink noise at a specified standard deviation. Internally, the module uses a white noise generator followed by an IIR filter.

### Arguments

* `NAME` - name of the module.
* `SR` - sample rate. This must be set since this is an output module.
* `NUMCHANNELS` - number of interleaved channels in each output pin.
* `BLOCKSIZE` - number of samples per output channel.

## Type Definition

N/A

## See Also

WhiteNoise, WhiteNoiseFract32, PinkFilter, Rand, Randi, RandiFract32, PinkNoiseFract32,

