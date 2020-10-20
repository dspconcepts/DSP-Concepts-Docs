---
description: Phase inverts signals (which is a fancy way of saying multiply by -1).
---

# Invert, InvertN

### Invert

This module inverts the phase of a signal. Inverting the phase is simply multiplying by -1, thus causing a phase shift. The variable `isInverted` is a Boolean which controls the behavior. When `isInverted`=1, the module multiplies the input signal by -1. When `isInverted`=0, the module passes the input signal through unchanged. Internally, the module uses a first order smoother to seamlessly switch between +1 and -1. The smoothing time of the switching process is set by the `smoothingTime` variable. Invert has only one input pin but it can accept multiple channels on that pin. The isInverted variable is applied to each channel.

#### Variables

![](../../../.gitbook/assets/0%20%2821%29.png)

* `isInverted` - When true, the incoming channels are multiplied by -1.
* `smoothingTime` - A time constant used in a 1st order filter to smooth the output as the module transitions from one state to the next.

### InvertN

This module is similar to the Invert module except that the `isInverted` variable is an array. When the second argument `NUMPINS` is 1, then the `isInverted` array is applied to each individual channel. For example isInverted\[1\] is applied to channel1, isInverted\[2\] is applied to channel2...

If NUMPINS is greater than 1, then size of the isInverted array equals NUMPINS. Now isInverted\[1\] applies to all channels on pin 1, isInverted\[2\] applies to all channels on pin 2, ...

#### Arguments

![](../../../.gitbook/assets/1%20%2822%29.png)

* `numPins` - Number of input pins for the module. When 1, `isInverted` array size will equal the number of channels on the wire connected to the input pin. When `numPins` is greater than 1, `isInverted` array size will equal `numPins` and each checkbox will invert all channels on one input pin.
* `numChannels` - An optional argument that specifies the initial number of channels in the wire. When the system is built, the number of channels is derived by pin propagation. This value is only for the initial behavior before the system is built. By default, `numChannels` = 2.
* `transpose` - When false, the inspector check boxes are aligned vertically. When true, the inspector checkboxes are aligned horizontally.

### Variables

![](../../../.gitbook/assets/2%20%2817%29.png)

* `smoothingTime` - A time constant used in a 1st order filter to smooth the output as the module transitions from one state to the next.

### Arrays

![](../../../.gitbook/assets/3%20%2817%29.png)

* `isInverted` - When true, the incoming channels are multiplied by -1.

## Examples

### Selective Inversion

When setting up a sound system there are many reasons for wanting to invert the phase of a signal. It may be a wiring error and the polarity of wires connected to the speaker have been reversed. Or it may be that the phase of a subwoofer needs to be inverted to better match the rest of the sound system. But for whatever reason we can think of to invert the phase this is a very handy module. The signal below shows a simple example of how to use these modules. Six signals have been derived from a sine wave, each has a unique amplitude. By examining the inspectors you will note that half of the signals have been inverted. This is shown in the sink display.

One nice feature of the InvertN inspector is that check boxes can be put in a horizontal line instead of a vertical line. It definitely helped to tidy up this signal flow.

![](../../../.gitbook/assets/4%20%2814%29.png)

## Inspector

### Invert

The inspector for the Invert module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to change the `isInverted` variable in real time.

![](../../../.gitbook/assets/5%20%287%29.png)

### InvertN

The inspectors for the InvertN module are shown below. They can be viewed by double clicking on the module in AWE\_designer. The inspectors allow you to change the `isInverted` variable array in real time.

By default, the inspector for the module has the control checkboxes organized in a vertical stack. By setting the optional third argument `TRANSPOSE` = 1 turns the control into a horizontal stack.

![](../../../.gitbook/assets/6%20%286%29.png) ![](../../../.gitbook/assets/7%20%287%29.png)

## Matlab Usage

`M=invert_module(NAME)`

Creates a module which inverts the phase of a signal \(that is, sets the gain equal to -1\). The change happens smoothly with a specified smoothing time.

Arguments:

* `NAME` - name of the module.

`M=invertn_module(NAME, NUMPINS, NUMCHANNELS, TRANSPOSE)`

Creates a module which inverts the phase of a signal \(that is, sets the gain equal to -1\). The change happens smoothly with a specified smoothing time.

Arguments:

* `NAME` - name of the module.
* `NUMPINS` - optional argument that specifies the number of input pins.
  * When `NUMPINS` is 1, the invert control applies to each individual channel within the input pin.
  * When `NUMPINS` is more than 1, the invert control applies to those input pins.
* `NUMCHANNELS` - An optional argument that specifies the initial number of channels in the wire. When the system is built, the number of channels is derived by pin propagation. This value is only for the initial behavior before the system is built. By default, NUMCHANNELS = 2.
* `TRANSPOSE` - an optional argument which controls how the inspector for the module is drawn. By default, if there are N channels then the inspector control is a vertical stack of N checkboxes. \(TRANSPOSE = 0 by default\). If TRANSPOSE = 1, then the inspector is drawn as a horizontal array of N checkboxes.

## Type Definition

```cpp
typedef struct _ModuleInvert
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 isInverted;                  // Boolean that specifies whether the signal
                                      // is inverted (=1) or not (=0).
   FLOAT32 smoothingTime;             // Time constant of the smoothing process.
   FLOAT32 currentGain;               // Instantaneous gain applied by the module.
                                      // This is also the starting gain of the
                                      // module.
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
} ModuleInvertClass;

typedef struct _ModuleInvertN
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure
   INT32 numControls;                 // Number of individual invert controls 
   FLOAT32 smoothingTime;             // Time constant of the smoothing process
   FLOAT32 smoothingCoeff;            // Smoothing coefficient.
   INT32* isInverted;                 // Boolean that specifies whether the signal
                                      // is inverted (=1) or not (=0). One per each
                                      // pin.
   FLOAT32* currentGain;              // Instantaneous gain applied by the module.
                                      // This is also the starting gain of the
                                      // module.
   FLOAT32* gain;                     // Target gain.
} ModuleInvertNClass;
```

## See Also

BlockConcatenate, BlockExtract, BlockFlip, InvertFract32, Reciprocal, ReciprocalFract32

