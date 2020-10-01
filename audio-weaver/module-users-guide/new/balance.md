---
description: Left-right balance control
---

# Balance

The Balance module smoothly scales left and right channels. The module has two multichannel input pins and two multichannel output pins. This allows the balance function to be applied to multiple channels with a single module. There is one balance variable that controls all balance functions. Internally the balance variable is automatically converted to target gains for left and right channels: `gainL` and `gainR`. Each left and right input channel is multiplied by `gainL` and `gainR` respectively.

The algorithm used to calculate `gainL` and `gainR` is called a sine/cosine panning law. Mathematically this panning law is defined by the following equations:

$$
gainL(balance)=cos((1+balance) * π/4) :- 1 ≤ balance ≤ 1
$$

$$
gainR(balance)=sin((1+balance) * π/4) :- 1 ≤ balance ≤ 1
$$

In order to avoid unwanted transients such as clicks and pops, there are 2 internal gain state variables `currentGainL` and `currentGainR` which are the outputs of the panning law algorithm. These gains are input into smoothing filters. The outputs of these filters are `gainL` and `gainR`. When balance is changed `gainL` and `gainR` will exponentially approach `currentGainL` and `currentGainR` with a time constant equal to `smoothingTime`. 

{% hint style="info" %}
We have only one time constant controlling both left and right channels.
{% endhint %}

### Variables

![](../../../.gitbook/assets/0%20%2813%29.png)

`balance` - determines the pan position from far left \(-1\) to far right\(1\).

`smoothingTime` - time constant to determine how fast changes to `balance` are seen at the outputs.

## Examples

To see how the panning law works, observe the signal flow below. The balance variable has been adjusted in the inspector to the hard left position. The Meter1 inspector shows that the level in the left channel is 0 dB which corresponds to a gain of 1.0 and the level in the right channel is for our purposes negative infinity which corresponds to a gain of 0.0.

$$gainL(-1) = cos((-1 + 1) * π/4) $$ which equals 1.0.

$$gainR(-1) = sin((-1 + 1) * π/4)$$ which equals 0.0.

![](../../../.gitbook/assets/1%20%2816%29.png)

The signal flow below shows the case where left and right are panned to the center position. The balance variable has been adjusted in the inspector to the center position. The Meter1 inspector shows that the level in the left and right channel is approximately -3 dB which corresponds to a gain of 0.7071.

\_\_$$gainL(0) = cos((0+1) * π/4)$$ which equals 0.7071

$$gainR(0) = sin((0+1) * π/4)$$ which equals 0.7071

 Which is what you would want when adding two uncorrelated signals together.

## ![](../../../.gitbook/assets/2%20%2813%29.png)

In the signal flow below, the balance variable has been adjusted in the inspector to the hard right position. The Meter1 inspector shows that the level in the left channel is for our purposes negative infinity which corresponds to a gain of 0.0 and the level in the right channel is 0 dB which corresponds to a gain of 1.0.

$$gainL(1) = cos((1+1) * π/4)$$ which equals 0.0

$$gainR(1) = sin((1+1) * π/4)$$ which equals 0.0

![](../../../.gitbook/assets/3%20%2816%29.png)

## Inspector

The inspector for this module can be viewed by double clicking on the module in AWE\_designer. The inspector allows you to adjust the balance variable in real time.

![](../../../.gitbook/assets/4%20%2811%29.png)

## Matlab Usage

To create a Balance module using Matlab scripts the function `balance_module` is called.

 `M=balance_module(NAME)`

###  Arguments

 `NAME` - name of the module.

## Type Definition

```cpp
typedef struct _ModuleBalance
{ 
    ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                       // structure
    FLOAT32 balance;                   // Left/Right Balance.
    FLOAT32 smoothingTime;             // Time constant of the smoothing
                                       // process.
    FLOAT32 currentGainL;              // Instantaneous gain applied to left
                                       // channel. This is also the starting
                                       // gain of the left channel.
    FLOAT32 currentGainR;              // Instantaneous gain applied to the
                                       // right channel. This is also the
                                       // starting gain of the right channel.
    FLOAT32 smoothingCoeff;            // Smoothing coefficient.
    FLOAT32 gainL;                     // Target gain left channel.
    FLOAT32 gainR;                     // Target gain right channel.     
} ModuleBalanceClass; 
```



## See Also

BalanceFract32, CrossFader

