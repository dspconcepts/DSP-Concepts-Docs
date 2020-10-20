---
description: >-
  Computes the maximum absolute value of all input channels on a
  sample-by-sample basis
---

# MaxAbs

This module is often used as part of dynamics processors, where the mono output of the MaxAbs module is used as the input to the LimiterCore or AutoAttackRelease module. MaxAbs can have multiple input pins. Each pin can have multiple channels. The number of channels on each pin can be different.

The module has 1 output channel with the same blockSize and sampleRate as the input. The module computes the following:

for\(i = 0; i &lt; blockSize; i++\)

 {

Output\[i\] = max\(abs\(chan1\[i\]\), abs\(chan2\[i\]\), …, abs\(chanN\[i\]\)\)

}

### Arguments

![](../../../.gitbook/assets/0%20%2815%29.png)

* `numIn` - The number of input pins.

## Examples

For examples of how MaxAbs is used in dynamics processing see the help file for AGC Limiter Core.

## Inspector

This module has no inspector.

## Matlab Usage

`MM=max_abs_module(NAME, NUMIN)`

This module calculates the maximum absolute value of all input channels on a sample-by-sample basis.

Arguments:

* `NAME` - name of the module.
* `NUMIN` - number of input pins.

## Type Definition

```cpp
typedef struct _ModuleMaxAbs
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure
} ModuleMaxAbsClass;
```

## See Also

Abs, AbsFract32, MaxAbsFract32, RunningMinMax, RunningMinMaxFract32

