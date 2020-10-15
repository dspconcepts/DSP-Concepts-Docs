---
description: >-
  The dB20 modules convert from linear units to decibels. The dB10 modules
  convert between energy and decibels.
---

# Db20, Db10, Db20 Approx, Db10 Approx, Undb10, Undb20, Undb10 Approx, Undb20 Approx

This help file discribes multiple modules. The first set of 4 modules does calculations uses the math library:

* `Db20` - Converts from linear units to decibels using the equation  **TODO**: fix equation Db20L=20\*log10L.
* `Db10` - Converts from energy to decibels using the equation **TODO**: fix equation Db10E=10\*log10E.
* `Undb20` - Converts from decibels to linear units using the equation **TODO**: fix equation Undb20db=10db/20
* `Undb10` - Converts from decibels to energy using the equation **TODO**: fix equation Undb10db=10db/10

Audio Weaver provides a second set of functions which approximate the 4 functions shown above. The approximation is performed using a truncated Taylor Series expansion and is faster on some targets.

* `Db20Approx` - Approximation to `Db20`.
* `Db10Approx` - Approximation to `Db10`.
* `Undb20Approx` - Approximation to `Undb20`.
* `Undb10Approx` - Approximation to `Undb10`.

## Examples

### Profile Comparison

In the profile table below the processing times of the approximated dB modules can be compared to the non-approximated dB modules. This test was run on a PC native target. As you can see the approximated dB modules are actually running slower. This is due to excellent intrinsics C libraries running on the Windows PC which take advantage of parallel processing hardware available on the PC. When running on other targets you might want to run a similar profile to see which dB module will be best for your target.

![](../../../.gitbook/assets/0%20%2819%29.png)

### Noise Floor Comparison

In the signal flow below the envelope of pink noise is used to determine the error noise floor of the Db20 module and the Db20 Approx module. From the Sink1 inspector we can see that error noise for Db20 to Undb20 is about -142 dB. The error noise for Db20 Approx to Undb20 Approx is about -66 dB.

![](../../../.gitbook/assets/1%20%2818%29.png)

## Inspector

There are no inspectors for these modules.

## Matlab Usage

`M=db10_module(NAME)`

Creates an Audio Weaver module that computes the function $$10*log10(x)$$ using the math library. This converts from energy to decibels.

#### Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=db10_approx_module(NAME)`

Creates an Audio Weaver module that computes the function 10\*log10\(x\) using a polynomial approximation. This converts from energy to decibels.

as

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=db20_module(NAME)`

Creates an Audio Weaver module that computes the function 20\*log10\(abs\(x\)\) using the math library. This converts from linear units to decibels.

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=db20_approx_module(NAME)`

Creates an Audio Weaver module that computes the function 20\*log10\(abs\(x\)\) using a polynomial approximation. This converts from linear units to decibels. The approximation is accurate to 7.984884e-003 dB.

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=undb10_module(NAME)`

Creates an Audio Weaver module that computes the function 10^\(x/10\) using the math library. This converts from decibels to energy units.

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=undb10_approx_module(NAME)`

Creates an Audio Weaver module that computes the function 10^\(x/10\) using a fast polynomial approximation This converts from decibels to linear units.

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=undb20_module(NAME)`

Creates an Audio Weaver module that computes the function 10^\(x/20\) using the math library. This converts from decibels to energy units.

Arguments:

* `NAME` - name of the module.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

`M=undb20_approx_module(NAME)`

Creates an Audio Weaver module that computes the function 10^\(x/20\) using a fast polynomial approximation. This converts from decibels to linear units.

Arguments:

* `NAME` - name of the module.

## Type Definitions

```cpp
typedef struct _ModuleDb10
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleDb10Class;

typedef struct _ModuleDb10Approx
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleDb10ApproxClass;

typedef struct _ModuleDb20
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleDb20Class;

typedef struct _ModuleDb20Approx
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleDb20ApproxClass;

typedef struct _ModuleUndb10
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleUndb10Class;

typedef struct _ModuleUndb10Approx
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleUndb10ApproxClass;

typedef struct _ModuleUndb20
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleUndb20Class;

typedef struct _ModuleUndb20Approx
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
} ModuleUndb20ApproxClass;
```

