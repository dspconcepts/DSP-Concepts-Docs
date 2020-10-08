---
description: Asymmetric clipper module with separate upper and lower clipping thresholds
---

# ClipAsym

Asymmetrical clipping, where the upper and lower thresholds are exposed as parameters. By setting `clipLower` and `clipUpper` thresholds equal, symmetrical clipping can be achieved. This can be used anytime a hard threshold is needed.

### Variables

![](../../../.gitbook/assets/0%20%2822%29.png)

* `clipLower` - Lower clipping threshold.
* `clippUpper` - Upper clipping threshold.

## Examples

### Complex Asymmetrical Clipping

Below is a simple example of how ClipAsym can be used to hard clip a complex signal.

![](../../../.gitbook/assets/1%20%2821%29.png)

The figure below shows the clipped complex signal.

![](../../../.gitbook/assets/2%20%2819%29.png)

## Inspector

The inspector for the ClipAsym module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to change clipUpper and clipLower variables in real time.

![](../../../.gitbook/assets/3%20%2811%29.png)

## Matlab Usage

 `M=clipasym_module(NAME)`

 Creates an asymmetric clip module for use with Audio Weaver.

 Arguments:

*  `NAME` - name of the module.

## Type Definition

```cpp
typedef struct _ModuleClipAsym
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
   // structure
   FLOAT32 clipLower; // Lower clipping threshold.
   FLOAT32 clipUpper; // Upper clipping threshold.
} ModuleClipAsymClass;
```

## See Also

ClipAsymFract32, ClipIndicator, ClipIndicatorFract32, SoftClip, SoftClipFract32

