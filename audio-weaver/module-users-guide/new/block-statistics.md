---
description: Computes statistics of a block of samples.
---

# Block Statistics

This module calculates statistics of the input signal on a block-by-block basis. The input pin can have an arbitrary number of interleaved channels and any block size, and the calculation occurs over all samples in the input pin. The output pin has a single channel and a block size of 1.

For each block, one of the following values is calculated: maximum, minimum, maximum absolute value, mean, RMS, standard deviation, variance, or average energy. The parameter `statisticsType` specifies which statistic is computed:

* 0=maximum,
* 1=minimum,
* 2=maximum absolute value,
* 3=mean,
* 4=RMS,
* 5=standard deviation,
* 6=variance,
* 7=average energy,
* 8=sum,
* 9=sum of squares.

The statistic is computed on a block-by-block basis. The module is stateless and the statistic is recomputed for each block.

An optional argument `outputValue` specifies whether the computed statistic should be output on an output pin. By default, `outputValue` is set to `NoOutputs` and the statistic is only stored internally. If `outputValue` is set to `OutputValue` then the value of the statistic is output on the first output pin as a floating-point value. If `outputValue` is set to `OutputValueIndex` then the module has two output pins. The first output pin is the statistic value and the second output pin is the zero based index at which the value occurs. Using `OutputValueIndex` only makes sense when `statisticType`=0 \(max\), 1 \(min\), or 2 \(maxAbs\).

By default, the sample rate of the output pin equals input pin sample rate / blocksize. There are some times when this computes the wrong sample rate and you can overwrite the output sample rate using the `outputSampleRate` argument. If `outputSampleRate` is empty then the output sample rate is set to sample rate / blockSize.

### Arguments

![](../../../.gitbook/assets/0%20%2818%29.png)

* `outputValue` - allows the user to choose no output pins, 1 output pin or 2 output pins.
* `outputSampleRate` - Allow the user to use a custom sample rate.

### Variables

![](../../../.gitbook/assets/1%20%2823%29.png)

* `statisticsType` - Allows the user to change which statistic is calculated.

## Description Block Statistics N

This block is similar to Block Statistics except the statistics are calculated on a channel by channel basis. Therefore the number of output channels equals the number of input channels with each output channel having a block size of 1.

## Examples

### Difference Between BlockStatistics and BlockStatisticsN

In the table below assume that each row represents a channel with block size of 8. Assume the variable `StatisticsType` set to `Mean`. The output of BlockStatistics will be a single channel with a buffer size of one. The output value will be 5.375. The equation used to compute this output is:

**TODO**: fix equation

For BlockStatisticsN the output will be 2 channels with a block size of one. The output values will be 3.625 and 7.125. The equations used to compute these values are:

**TODO**: fix equations

| 2 | 3 | 7 | 5 | 3 | 2 | 6 | 1 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 7 | 7 | 8 | 1 | 10 | 9 | 10 | 5 |

Mean of row 1 = 3.625; Mean of row 2 = 7.125; Mean of both rows = 5.375

### Find Frequency of a Sine Wave Using Block Statistics

The figure below shows a signal flow that uses BlockStatistics to calculate the frequency of a sine wave. The BlockStatistic `statisticsType` variable is set to `Maximum`. The input to the FFT module has a sample rate and a block size of 16384. This yields a frequency resolution of 48000/16384 = 2.9397 Hz/bin. In the block diagram below, the BlockStatistic module outputs the sample index at which the maximum occurs. This is multiplied to 2.9397 to obtain the frequency in Hz of the peak. The result closely matches the frequency of the sine wave generator and the 0.98 error is due to the frequency resolution of the FFT.

![](../../../.gitbook/assets/2%20%2820%29.png)

## Inspector

The inspector for the BlockStatisics module is shown below and can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to select the `statisticsType` variable and view the output meter in real time.

![](../../../.gitbook/assets/3%20%2818%29.png)

The inspector for the BlockStatisicsN module is shown below and can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to change the `statisticsType` variable in real time.

![](../../../.gitbook/assets/4%20%2812%29.png)

## Matlab Usage

To create a Block Statistics module using Matlab scripts the function block\_statistics\_module is called.

 `M=block_statistics_module(NAME, OUTPUTVALUE, SAMPLERATE)`

This module calculates statistics of the input signal on a block-by-block basis. The type of statistics calculated is determined by the statisticsType parameter. The computed value is stored in the internal state variable instantaneousValue.

 Arguments:

* `NAME` - name of the module.
* `OUTPUTVALUE` - Integer value which specifies whether the module has an output pin.
  * If `OUTPUTVALUE`=0,then the module has no output pin.
  * If `OUTPUTVALUE`=1 \(the default\), then the `instantaneousValue` variable is output as a single sample on the first output pin.
  * If `OUTPUTVALUE`=2 then the module has a second output pin containing the index at which the value occurred. Outputting the `index` only makes sense for Max, Min, and MaxAbs statistics. In all other cases, `index = -1`.
* `SAMPLERATE` - optional argument which specifies the output sample rate of the module. By default, this is empty and the output sample rate is computed as `inputSampleRate` / `blockSize`. In some cases, this sample rate is wrong and you can override it with this argument.

To create a Block StatisticsN module using Matlab scripts the function block\_statisticsn\_module is called.

 `M=block_statisticsn_module(NAME, OUTPUTINDEX, SAMPLERATE)`

This module calculates statistics of the input signal on block-by-block basis per channel. This is similar to block\_statistics\_module.m but has a multichannel output. The type of statistics calculated is determined by the statisticsType parameter. An optional per-channel index output for Max, Min, and MaxAbs statistics are available.

 Arguments:

The arguments for this module are the same as the Block Statistics module above.

## Type Definition

```cpp
typedef struct _ModuleBlockStatistics
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
    INT32 statisticsType;              // Type of statistics calculated.
    FLOAT32 instantaneousValue;        // Instantaneous output value.
} ModuleBlockStatisticsClass;

typedef struct _ModuleBlockStatisticsN
{
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
     INT32 statisticsType;             // Type of statistics calculated.
} ModuleBlockStatisticsNClass;
```

## See Also

Averager, BlockMedian, BlockStatistics, RMS, RMSN, RunningMinMax, RunningStatistics, SampleStatistics, SbRMS, SubblockStatistics

