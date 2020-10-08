---
description: Performs addition of signals.
---

# Adder

The adder has a variable number of input pins and a single output pin. Each pin can have multiple channels. The module operates in two different modes:

**Normal** - each input pin has N channels and the output pin will also have N channels.

**Single Channel Output** - each input pin can have a different number of channels. All channels are summed to a mono output.

All input pins must have the same block size and sample rate. The block size and sample rate of the output pin will match the input pins.

### Arguments

![](../../../.gitbook/assets/0%20%2811%29.png)

`numIn` - The number of input pins.

`oneChannelOutput` - This is a boolean that determines the mode of the adder.

1. False - The module functions in “Normal” mode.. All of the input pins must have the same number of channels. The output pin will then have the same number of channels as the input pins and be formed as the sum of all input channels.
2. True - The module functions in “Single Channel Output” mode. The input pins can each have a different number of channels. The output is then a one channel signal that is formed as the sum over all input channels.

Note, when summing mono signals it is better to use “Normal” mode. Enabling “Single Channel Output” will give you the same result but will be computationally slower.

## Examples

The following examples show the difference between the 2 modes of the adder module.

### Example 1 - Normal Mode.

 In the figure below, the argument `oneChannelOutput` is set to **false**. Therefore each input pin must have the same number of channels. In this case each input pin has 3 channels. The resulting output pin also has 3 channels. The constant values for the channels of Input pin1 are 1, 2 and 3. The constant values for the channels of input pin2 are 4, 5 and 6. As shown in the sink display plot, there are 3 channels in the output pin.

```text
Output Channel1 = input pin1 channel1+ input pin2 channel1

Output Channel2 = input pin1 channel2 + input pin2 channel2

Output Channel3 = input pin1 channel3 + input pin2 channel3
```



![](../../../.gitbook/assets/1%20%2811%29.png)

### Example 2 - Single Channel Output

In the figure below the `oneChannelOutput` argument is set to **true**. Therefore each input pin can have a different number of input channels. In this case input pin1 has 3 input channels and input pin 2 has 2 input channels. The resulting output pin also has 1 channel. The constant values for the channels of Input pin1 are 1, 2 and 3. The constant values for the channels of input pin2 are 4 and 5. As shown in the sink display plot, there is 1 channel in the output pin.

```text
Output Channel 1 = input pin1 channel1 + input pin1 channel2 + input pin1 channel3 +
input pin2 channel1 + input pin2 channel2
```

![](../../../.gitbook/assets/2%20%2811%29.png)

## Matlab Usage

To create an adder module using Matlab scripts the function adder\_module.m is called.

The function header is:

`M=adder_module(NAME, NUMIN, ONECHANNELOUTPUT)`

### Arguments:

`NAME` - name of the module. Must be a string, i.e. `adder1`.

`NUMIN` - number of input pins.

`ONECHANNELOUTPUT` - boolean value which determines if multi-channel input signals are collapsed to a single output channel.

## Type Definition

```cpp
typedef struct _ModuleAdder
{
    ModuleInstanceDescriptor instance;      
    INT32 oneChannelOutput;            
} ModuleAdderClass;
```



## See Also

AdderFract32, AdderInt32, SquareAdd, SquareAddFract32, SumDiff, SumDiffFract32, SumDiffInt32, Subtract

