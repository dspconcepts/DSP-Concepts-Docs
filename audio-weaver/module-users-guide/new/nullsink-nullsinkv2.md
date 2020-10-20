---
description: Sink module which performs no computation and discards the input.
---

# NullSink NullSinkV2

These modules serve as terminators for unconnected wires in the layout. They work great if your signal flow has an orphan pin that needs to be connected before the signal flow can run. The inputs can be multichannel.

NullSinkV2 has the added feature of being able to have multiple input pins. Each input pin can have a different number of channels.

The NullSink and NullSinkV2 modules essentially do no computation. The processing function does an immediate return from function call. The Sink module, on the other hand, has an internal buffer equal in size to the input wire. The Sink module copies the input wire data to the internal buffer \(this takes some processing\), and the Sink inspector displays this waveform.

### Arguments NullSinkV2

![](../../../.gitbook/assets/0%20%2836%29.png)

* `numPins` - Defines the number of multichannel input pins.

## Examples

### Orphan Pin

The signal flow below shows a simple implementation of a linear predictor. Once the signal flow has been debugged and is working correctly there is no need to monitor the errorSignal output pin. Processing bandwidth can be saved by connecting this orphaned pin to a NullSink. If there are more than orphaned pin, module count can be decreased by using NullSinkV2.

![](../../../.gitbook/assets/1%20%2826%29.png)

### Multiple Input Pins

In the signal flow below, there is a PinkNoise module with 6 channels of output. There is also a SineSmoothedGen module with 1 channel of output. Note that both sink modules can accept the 6 channels of pink noise. Also note that the NullSinkV2 module’s input pins can have a different number of channels for each pin.

![](../../../.gitbook/assets/2%20%2826%29.png)

## Inspector

There are no inspectors for these modules.

## Matlab Usage

`M=null_sink_module(NAME)`

This module serves as an end for unconnected wires in the layout. The processing function is called but performs no actual computation.

### Arguments

* `NAME` - name of the module.

 `M=null_sink_v2_module(NAME, NUMPINS)`

This module serves as an end for unconnected wires in the layout. The processing function is called but performs no actual computation. This module supports a variable number of input pins.

### Arguments

* `NAME` - name of the module.
* `NUMPINS` - Optional: number of inputs. Default = 1 input.

## Type Definition

```cpp
typedef struct _ModuleNullSink
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure
} ModuleNullSinkClass;

typedef struct _ModuleNullSinkV2
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure
} ModuleNullSinkV2Class;
```

## See Also

BooleanSink, FileSink, InterleavedSink, Sink, SinkFract32, SinkInt, TriggeredSink Source

