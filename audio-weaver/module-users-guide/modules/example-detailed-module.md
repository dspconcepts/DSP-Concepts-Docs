---
description: Flexible copying and interleaving of audio channels.
---

# Router

## Description

The Router module is one of the most useful and frequently used modules.  It copies channels between wires using a routing table.  The module arguments specify the number of input pins and the number of output channels \(N\).  The module always has a single output pin and copies one input channel \(from any pin\) to each of the N output channel.

An inspector for a Router module with 4 output channels is shown below.  Each row in the table corresponds to a different output channel.  The first output channel is input channel 6; the second output channel is input channel 7; the third output channel is input channel 8; and the fourth output channel is input channel 5.

![](../../../.gitbook/assets/image%20%2889%29.png)

The Router module operates on all 32-bit data types \(floating-point, fract32, and integer\).  The module supports changing the channel mapping at run-time. Keep in mind that this module does not use smoothly and you may get audible pops and clicks.  The RouterSmoothed module has the same functionality but includes smoothing.

{% hint style="info" %}
Note that the input channels shown in the routing table are updated during pin propagation.  If you do not see the correct channels available, you will need to fullly wire your block diagram and then Propagate Changes.
{% endhint %}

## Examples

### Selecting a Single Channel

Suppose that you have a stereo signal with interleaved left and right components. To extract the left channel, you could use a standard Deinterleaver with a NullSink on the right channel output:

![](../../../.gitbook/assets/image%20%2842%29.png)

This is wasteful because it requires 3 modules to do what a single router module can do. Create a Router with 1 input pin and 1 output channel.  First set the module arguments as follow:

![](../../../.gitbook/assets/image%20%2876%29.png)

Then set the channel map as:

![](../../../.gitbook/assets/image%20%2834%29.png)

The system only requires a single Router module now:

![](../../../.gitbook/assets/image%20%2863%29.png)

### Extracting L and R from a 5.1 channel signal

Supposed you have a 6 channel wiring corresponding to a 5.1 channel signal.  Assume that the channels are ordered: L, R, C, LFE, Ls, Rs.  You can extract the L and R signals using a single Router module.  Set the number of input pins to 1 and the number of output channels to 2.  Then configure the routing table as:

![](../../../.gitbook/assets/image%20%2875%29.png)

### Swapping Left and Right Channels

The output channels do now have to be in the same order as the inputs. For example, suppose you have a stereo signal and you want to swap the order of the left and right channels. Create the Router module with 1 input pin and 2 output channels. Then set:

![](../../../.gitbook/assets/image%20%2861%29.png)

### LFE Processing

Suppose you have a 5.1 signal with channels ordered: L, R, C, LFE, Ls, Rs and you want to processing the LFE channel separately.  You could do this with Router modules as shown below:

![](../../../.gitbook/assets/image%20%2862%29.png)

Router1 removes the LFE channel and is configured as follows:

![](../../../.gitbook/assets/image%20%2886%29.png)

Router2 extracts the LFE channel:

![](../../../.gitbook/assets/image%20%2857%29.png)

Router 3 is configured to have 2 input pins and 6 output channels, and its routing table is configured as:

![](../../../.gitbook/assets/image%20%2831%29.png)

## Matlab Usage

The Router module is implemented by the function router\_module.m.  The function header is:

`M = router_module(NAME, NUMINPIN, NUMOUTCHANNEL)`

where

NAME - name of the module

NUMINPIN - number of input pins

NUMOUTCHANNEL - number of output channels.

The module has an array channelIndex of length N.  channelIndex\[k\] specifies the mapping of input channels \(pin and channel\) to output channel k.  Each 32-bit integer value is packed as:

channelIndex = \(pinNum &lt;&lt; 16\) + \(channelNum\)

or alternatively,

channelIndex = \(pinNum \* 65536\) + \(channelNum\).

pinNum and channelNum are both zero-based.  If channelIndex is out of range, then the corresponding output channel will be muted.  If you desire to mute a specific output channel, then set channelIndex = -1.

## Type Definition

`typedef struct _ModuleRouter`

`{`

    `ModuleInstanceDescriptor instance;`

    `INT32* channelIndex;`

`} ModuleRouterClass;`

## See Also

RouterSmoothed, Interleaver, Deinterleaver, MultiplexorV2

