---
description: Butterworth filter with standard and high precision implementations
---

# Butterworth, ButterworthHP

The Butterworth modules can be used to create low pass, high pass, and all pass filters. Both modules operate on multi-channel 32-bit floating-point data.

The Butterworth filter uses a standard cascade of second order biquads implemented using floating-point arithmetic. The ButterworthHP still uses floating-point arithmetic but achieves higher precision by using a state space implementation.

### Arguments

![](../../../.gitbook/assets/0%20%2823%29.png)

`order` - Select the order of the Butterworth filter. Filter orders of 1 to 10 can be selected. This argument cannot be changed in real time.

Note that only odd order allpass filters are supported. If you try and build an even order allpass, you will get a warning message.

### Variables

![](../../../.gitbook/assets/1%20%2814%29.png)

`filterType` - Select which type of filter is designed. This variable can be changed at run-time.

* filterType=0 designs a low pass filter.
* filterType=1 designs a high pass filter.
* filterType=2 designs an allpass filter.

`cutoff` - Cutoff frequency of Butterworth filter.

## Examples

### Butterworth vs ButterworthHP

Let’s investigate the processing self noise of two filters. In the signal flow below a 20 Hz sine wave is filtered by a LP Butterworth module and a LP ButterworthHP module, both with a cuttoff of 40 Hz. The results of both filters are then filtered by a HP ButterworthHP with a cutoff frequency of 200 Hz. In theory, if the filters were perfect there should be no energy left in both signals. But because of quantization errors in filter coefficients and processing techniques, the noise floor of the processed signals will increase. This is shown by measuring the remaining power in both channels. Looking at the Meter1 inspector the noise floor in channel 1 is -108.31 dB, the noise floor in channel 2 is -122.31. Using the butterworthHP module has lowered the noise floor by 14 dB!

![](../../../.gitbook/assets/2%20%2814%29.png)

### Profiling ButterWorth vs ButterWorthHP

The table below was created by the profiling tool in awe\_designer. The highlighted lines show the performance numbers for both LP filters shown in the figure above. As you can see there is a performance hit speed and memory required for the higher performing High Precession filter.

![](../../../.gitbook/assets/3%20%2810%29.png)

## Inspector

The inspector for the Butterworth module is shown below. It can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to select the `filterType` variable and change the `cutoff` frequency in real time.

![](../../../.gitbook/assets/4%20%2810%29.png)

## Matlab Usage

 `SYS=butter_filter_module(NAME, ORDER)`

 Creates a Butterworth high-pass or low-pass filter.

 Arguments:

* `NAME` - name of the module.
* `ORDER` - filter order.

 `SYS=butter_filterHP_module(NAME, ORDER)`

 Creates a Butterworth high-pass or low-pass filter. This version uses a high precision implementation.

 Arguments:

*  `NAME` - name of the module.
*  `ORDER` - filter order.

## Type Definition

```cpp
typedef struct _ModuleButterworthFilter
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module
 // instance structure
 INT32 filterType; // Specifies what type of filter is designed
 // (lowpass=0, highpass=1, allpass=2).
 INT32 order; // Order of the filter.
 FLOAT32 cutoff; // Cutoff frequency of the filter in Hz
 awe_modBiquadCascadeInstance *filt; // Cascade of second order Biquad filters
} ModuleButterworthFilterClass;
typedef struct _ModuleButterworthFilterHP
{
 ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
 // structure
 INT32 filterType; // Specifies what type of filter is designed
 // (lowpass=0, highpass=1, allpass=2).
 INT32 order; // Order of the filter.
 FLOAT32 cutoff; // Cutoff frequency of the filter in Hz
 awe_modBiquadCascadeHPInstance *filt;// Cascade of second order Biquad filters with
 // high precision implementation
} ModuleButterworthFilterHPClass;

```

## See Also

ButterworthFilterFract32, ButterworthFilterHP, SOFControl Biquad FIR

