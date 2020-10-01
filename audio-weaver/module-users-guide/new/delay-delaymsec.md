---
description: Time delay in which the delay is specified in samples or milliseconds.
---

# Delay, DelayMsec

### Delay

A delay module, where the delay is specified as an integer number of samples. The delay is implemented such that delays as short as 0 samples can be realized. The module has one multichannel input and one multichannel output. The module can operate on multiple channels and applies the same delay to each channel.

At construction time, specify the maximum delay \(`maxDelay`\) of the module. The variable `maxDelay` determines the size of the internal state buffer. The size of the internal state buffer equals \(`maxDelay`+1\)\*`numChannels`. In addition, the instantaneous delay is specified through the interface variable, `currentDelay`. `currentDelay` is in the range of \[0 `maxDelay`\] and can be changed at run-time. Note that the module is not smoothly updating and you may get audible clicks when adjusting `currentDelay` at run-time.

If you need finer delay resolution than 1 sample, then use the `FractionalDelay` module.

#### Arguments

![](../../../.gitbook/assets/0%20%2817%29.png)

* `maxDelay` - The maximum delay in samples.
* `memHeap` - Memory heap on the target where the state buffer is stored. See Matlab Usage below to see the different options.

#### Variables

![](../../../.gitbook/assets/1%20%2813%29.png)

* `currentDelay` - Set to the desired delay in samples. It must be less than or equal to `maxDelay`.

### DelayMsec

A delay module, where the delay is specified in milliseconds. The module has one multichannel input and one multichannel output. The module can operate on multiple channels and applies the same delay to each channel.

At construction time, specify the maximum delay time \(`maxDelay`\) of the module. The variable `maxDelay` determines the size of the internal state buffer. Let maxSamples equal the number of samples needed to implement a delay of maxDelay mSec. Then maxSamples = `maxDelay`/1000 \* sampleRate. The size of the internal state buffer equals \(maxSamples+1\)\*numChannels. In addition, the instantaneous delay is specified through the interface variable, `currentDelay`. `currentDelay` is in the range of \[0 maxDelayTime\] and can be changed at run-time. Note that the module is not smoothly updating and you may get audible clicks when adjusting the currentDelay at run-time.

A word of warning. When your target is a DSP with limited memory, `maxDelay` can eat up a lot of precious memory. For example when `currentDelay` is set to 20 samples and `maxDelay` is set to 100 samples, you will be wasting 80 memory locations per channel.

#### Arguments

![](../../../.gitbook/assets/2%20%2818%29.png)

* `maxDelay` - Maximum time delay in milliseconds.
* `memHeap` - Memory heap on the target where the state buffer is stored. See Matlab Usage below to see the different options.

#### Variables

![](../../../.gitbook/assets/3%20%2812%29.png)

* currentDelayTime - Set to the desired delay in samples. It must be less than or equal to `maxDelay`. Note that currentDelayTime is rounded towards zero to the closest integer sample.

## Examples

### Speaker Time Alignment

Consider yourself sitting in the driver’s seat of a car. It is an American car so you are sitting on the left side of the car. There is a woofer on the left side of the car. It is 2 feet from your left ear. There is also a woofer on the right side of the car. It is 5 feet away from your right ear. Since sound travels at 1126 feet/second it will take the sound from the left speaker 1.776 msecs to reach your left ear. It will take sound from the right speaker 4.44 msecs to reach your right ear. In order to time align the left and right speakers to the driver seat position it will be necessary to delay the left speaker by 4.44 - 1.78 = 2.66 msec. This is depicted in the signal flow below.

![](../../../.gitbook/assets/4%20%2813%29.png)

### How Many Samples are Needed for a 2.66 msec Delay?

Have you ever wondered what happens when the variableDelay time ends up being between two samples. For example in the above example the time alignment delay was set at 2.66 msecs. We can calculate the number of samples required using the following equation:

_samples_ = _variableDelayTime_/1000 \* _sampleRate_

Using this equation the number of samples equals 127.68. If we round this number to 128 samples, we can see in the graph below that the two channels are not time aligned. If we round towards zero the two channels are time aligned. Therefore the correct equation for calculating the number of samples used by the DelayMsec module is:

_samples_ + _fix_\(_variableDelayTime_/1000 \* _sampleRate_\)

![](../../../.gitbook/assets/5%20%288%29.png)

## Inspector

The inspectors for the Delay and DelayMsec modules are shown below. They can be viewed by double clicking on the modules in AWE\_designer. The inspectors allows you to change the `currentDelay` variables in real time.

![](../../../.gitbook/assets/6%20%287%29.png)

## Matlab Usage

`M=delay_module(NAME, MAXDELAY, MEMHEAP)`

Creates an integer time delay for use in the Audio Weaver environment.

Arguments:

* `NAME` - name of the module.
* `MAXDELAY` - maximum delay, in samples.
* `MEMHEAP` - specifies the memory heap to use to allocate the main state buffer. This is a string and follows the memory allocation enumeration in Framework.h. Allowable values are:
  * `'AWE_HEAP_FAST'` - always use internal DM memory.
  * `'AWE_HEAP_FASTB'` - always use internal PM memory.
  * `'AWE_HEAP_SLOW'` - always use external memory \(the default\).
  * `'AWE_HEAP_FAST2SLOW'` - use internal memory. If this fails then use external memory \(the default\).

`SYS=delaymsec_module(NAME, MAXDELAY, MEMHEAP)`

Creates a delay module in which the delay times are specified in milliseconds.

Arguments:

* `NAME` - name of the module.
* `MAXDELAY` - maximum delay of the module, in milliseconds.
* `MEMHEAP` - specifies the memory heap to use to allocate the main state buffer. This is a string and follows the memory allocation enumeration in Framework.h. Allowable values are:
  * `'AWE_HEAP_FAST'` - always use internal DM memory.
  * `'AWE_HEAP_FASTB'` - always use internal PM memory.
  * `'AWE_HEAP_SLOW'` - always use external memory.
  * `'AWE_HEAP_FAST2SLOW'` - use internal memory. If this fails then use external memory \(the default\).
  * `'AWE_HEAP_FASTB2SLOW'` - use internal memory. If this fails then use external memory.

 Internally, this uses a sample-based delay and builds an interface on top of it.

## Type Definition

```cpp
typedef struct _ModuleDelay
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 // structure
 INT32 maxDelay; // Maximum delay, in samples. The size of the
 // delay buffer is (maxDelay+1)*numChannels.
 INT32 currentDelay; // Current delay.
 INT32 stateIndex; // Index of the oldest state variable in the
 // array of state variables.
 INT32 stateHeap; // Heap in which to allocate memory.
 FLOAT32* state; // State variable array.
} ModuleDelayClass;
typedef struct _ModuleDelayMsec
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 // structure
 FLOAT32 maxDelayTime; // Maximum delay, in milliseconds.
 FLOAT32 currentDelayTime; // Current delay.
 INT32 stateHeap; // Heap in which to allocate state buffer
 // memory.
 awe_modDelayInstance *delay; // Time delay in which the delay is specified
 // in samples.
} ModuleDelayMsecClass;

```

## See Also

AllpassDelay, AllpassDelay16, AllpassDelay16Fract32, AllpassDelayci, AllpassDelayFract32, AllpassDelayi, BlockDelay, Delay16, DelayciFract32, DelayInterp, DelayInterpFract32, DelayNChan, DelayNChanMsec, DelayNTap, DelayNTap16Fract32, DelayReader, DelayStateWriter, DelayStateWriter16, FractionalDelayFract32

