---
description: General purpose multiplexor
---

# MultiplexorV2

Smoothly varying multiplexor module with fade time between input transitions and an optional control pin \(pin 1 when enabled\). The control pin is an integer value in the range 0 to N-1, where N is the number of audio inputs. If the control value is outside of the allowable range it is clipped. The first sample value of the control pin is used as the index for the multiplexor; all other values are ignored. Meaning the first sample of each block is used to determine the state of the multiplexor. The MultiplexorV2 module can only switch at the beginning of each block.

Each audio input pin must have the same number of channels, sample rate, block size and data type.

This module has built-in smoothing to minimize audible clicks when transitioning between two inputs. The smoothing is performed using a first order smoother and the variable smoothingTime controls the transition rate. It takes about 3 x smoothingTime for the complete transition to occur.

The module can also be configured to output zeros between transitions. The variable fadeTime specifies the silence period, in msec, when transitions occur.

### Arguments

![](../../../.gitbook/assets/0%20%2827%29.png)

* `isControl` - When true the first input pin is a control input. The wire connected to this pin does not need to be the same block size and sample rate as the input data pins. While this provides a somewhat asynchronous decoupling between the control pin and the data pins, be warned that in the example below that period and on time of the signal is a bit tricky to predict. The control pin can also have more than one channel. But the index will be the first sample of the first channel.
* `numInPins` - The number of input pins to be multiplexed.

### Variables![](../../../.gitbook/assets/1%20%2834%29.png)

* `index` - Used to switch between input pins when there is no external control pin. It can be changed when there is a control pin but the control pin takes priority.
* `smoothingTime` - Time constant of the first order smoothing process. In msec.
* `fadeTime` - When the index signal changes, the output pin is blanked for fadeTime msecs. It is important to note that this blanking happens before the smoothing process.

## Examples

### Multiplexor with Control Input and Fade Time

The signal flow below shows a MultiplexorV2 module. The `isControl` argument has been set to true so input pin1 is now a control input. The control signal is a square wave with a period of 20 msec and on time of 10 msec. The 2 data inputs are a DC source that has been set to a value of 0.2, and a pulse generator whose amplitude ranges from 0 to 1.

We will be looking at the sink plot as the `smoothingTime` and `fadeTime` variables are changed.

![](../../../.gitbook/assets/2%20%2832%29.png)

The plot below shows the output of the MultiplexorV2 module. Note the waveform is as we would think. The 1 msec pulses are on for 10 msec and the DC constant of 0.2 is on for 10 msec.

![](../../../.gitbook/assets/3%20%2819%29.png)

The plot below shows what happens when the smoothing time constant is set to 1 msec. We can see the envelope of the smoothing circuit. As predicted, It takes approximately 3 msec to settle.

![](../../../.gitbook/assets/4%20%2817%29.png)

The next plot shows what happens when the `fadeTime` variable is set to 5 msec. \(This is probably a larger blanking time than would normally be used but is used here to emphasize the effect.\) Notice that 5 msecs of zeros is inserted each time the control signal changes.

![](../../../.gitbook/assets/5%20%2813%29.png)

For the final plot below the `smoothingTime` variable is set to 1 msec and the `fadeTime` variable is set to 5 msecs. From the plot above it is seen that it’s graph has been filtered using the 1 msec time constant. You should note the order that things happen here. The `fadeTime` blanking happens before the smoothing filter.

![](../../../.gitbook/assets/6%20%2811%29.png)

## Inspector

The inspector can be viewed by double clicking on the MultiplexorV2 module in AWE\_designer. The inspector allows you to view and change the `index` in real time. Please note that if the argument `isControl` is true then the inspector cannot be viewed.

If the MultiplexorV2 has 2 pins to select from, then the inspector will be a checkbox. Unchecked selects pin 0; checked selects pin 1.

![](../../../.gitbook/assets/7%20%288%29.png)

If the MultiplexorV2 has more pins, then instead of a checkbox, you will have a droplist:

![](../../../.gitbook/assets/8%20%287%29%20%281%29.png)

## Matlab Usage

`M = multiplexor_v2_module(NAME, ISCONTROL, NUMINPINS)`

Creates a multiplexor module that seamlessly switches between inputs. The module has multiple input pins and switches from one input to another. The optional control pin is the first pin \(when enabled\) and determines which of N inputs is selected. The input pin is an integer value in the range 0 to N-1.

Arguments:

* `NAME` - name of the module.
* `ISCONTROL` - enables or disables the control pin.
* `NUMINPINS` - number of input pins.

All input pins must have the same block size and number of channels.

Note, this version implements an optional smooth crossfade between inputs.

## Type Definition

```cpp
typedef struct _ModuleMultiplexorV2
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 index;                       // Specifies which input pin to route to the
                                      // output. The index is zero based.
   FLOAT32 smoothingTime;             // Time constant of the smoothing process.
   FLOAT32 fadeTime;                  // Silence time in crossfade
   INT32 isControl;                   // Indicates if the index is controlled by an
                                      // external signal.
   FLOAT32 smoothingCoeff;            // Smoothing coefficient
   FLOAT32 oldSrcGain;                // Instantaneous gain being applied to the old
                                      // source. This is an internal variable used
                                      // in the smoothing process.
   FLOAT32 newSrcGain;                // Instantaneous gain being applied to the new
                                      // source. This is an internal variable used
                                      // in the smoothing process.
   INT32 fadeState;                   // State variable for determining when to
                                      // transitioning between inputs.
   INT32 fadeStateInit;               // Derived from fadeTime, determines number of
                                      // blocks to wait before switching input pins.
   INT32 oldIndex;                    // This is the index that is currently being
                                      // used.
   INT32 newIndex;                    // This is the index being transitioned to.
} ModuleMultiplexorV2Class;

```

## See Also

 Multiplexor, MultiplexorV2Fract32, Route,r RouterSmoothed, RouterSmoothedFract32, SampleMultiplexorControl

