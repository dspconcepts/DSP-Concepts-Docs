# Module Users Guide

## Introduction

This document is an overview of the audio modules included in Audio Weaver. It is intended for training users how to get the most out of the module library by providing a high-level overview with in depth examples of the modules. Modules build up sophisticated audio functions and systems in a matter of clicks, similar to circuit design or using lego blocks. Further information is available in the detailed HTML documentation provided with the Audio Weaver installer. For more information about a specific module, run Audio Weaver Designer and right click a module to view its “Help” file.

### How to use this guide

This guide is separated into sections as outlined in the table of contents: Chapter 2 starts us off with basic module concepts. It also discusses designer workflow including the module properties window and adjusting parameters. Chapter 3 explains each module folder in the browser, as well as how to pick the correct module as many of the modules are similar. To be clear about the differences between the modules, the end of each section in Chapter 3 includes a table that summarizes the differences between the modules in that folder. Chapter 4 provides in depth examples of common processing techniques and algorithms so the user can have a taste of the capabilities of Audio Weaver. **Usage tips are bolded throughout this guide for ease of reference.**

### Other Audio Weaver Documents

This document describes the Audio Weaver modules and module library. The MATLAB scripting interface is described in [Audio Weaver Matlab API](../../matlab-api.md). Read more about the graphical designer in [Audio Weaver Designer User’s Guide](../../audio-weaver-designer/users-guide.md).

{% page-ref page="../../audio-weaver-designer/users-guide.md" %}



### Module Library Organization

Audio samples are represented as 32-bit values. Several different data types are available:

* **Float32** – standard 32-bit floating-point with 1 sign bit, 8 exponent bits, and 23 mantissa bits.
* **Int32** – Standard twos complement 32-bit integer. The signed values are in the range $$[-2^{31},+(2^{31})]$$. 
* **Fract32** – Fractional representation where values are scaled in the range $$[-1 +1)$$ . This is standard integer representation with an implied scale factor of $$2^{-31}$$ .  



![](../../.gitbook/assets/2.png)

**At the top of the module browser, there is a checkbox to filter based on the different module data types** \(_see above_\). This will filter out modules for hardware that operates on specific sample data type \(fract32 for fixed-point\). The integer module libraries are typically used for control operations and work on both fixed-point and floating-point targets. The type convert module allows any data type to transfer into the others. This may be destructive if converting to a type with lower resolution.

### Module Browser overview

Audio Weaver modules are organized into separate browser folders based on their function. The folders are arranged in alphabetical order titled with a short description of the contents. A search bar can be found directly above the browser window. **To use the search bar, type the name or some common tag for the wanted module and press enter.**

## Basic Module Concepts

Audio Weaver Designer has two modes: Design mode and Tuning mode. In Design mode, design signal flow: by adding modules, making connections, and setting parameters. Tuning mode occurs when the designed layout is run. **The layout can only be run if all modules have valid connections. Run the layout by clicking the play button at the top or by right clicking the canvas and selecting “Build and Run.”** Tuning mode allows for parameter changes only: no changes to the wiring or structure of the block diagram.

### Viewing Module Properties

![](../../.gitbook/assets/assets_dsp-concepts_-m2bisi4gerfijm66kfl_-m2bkmkkrbyodjb4gbz7_3.png)

An audio module gets instantiated when dragged to the canvas from the Module Browser, located on the left hand side of the canvas. **Module** _**arguments**_ **can only be changed in Design mode and affect memory allocation, pins, and wiring. Module** _**variables**_ **are tunable parameters which can be changed in either mode.** Arguments tend to define memory allocation for the module, which can’t be changed during runtime. For example, in an FIR filter, the length of the filter \(number of taps\) is specified as an argument. This length affects memory allocation and as a result setting these arguments is only allowed in Design mode. The FIR coefficients \(a variable\) can change at any time.

After a module is instantiated, change its arguments and variables by right-clicking and selecting “View Properties” \(see above\) which will open up the properties manager at the bottom \(see below\).

![](../../.gitbook/assets/4%20%282%29.png)![](../../.gitbook/assets/5%20%285%29.png)

In the Matlab code for an FIR module, the module properties maps directly to the function arguments: `M=fir_module(NAME, L)`

### Tunable Variables and Inspectors

![](../../.gitbook/assets/6%20%283%29.png)

Most modules also have an associated _Inspector._ The inspector allows changing of the module’s tunable parameters. Inspectors can be used in Design mode and Tuning mode by double-clicking on a module, or by right-clicking and selecting “Inspector.” The inspector for an FIR filter holds the filter coefficients \(see above\). These parameters can also be changed under the “Variables” tab in the module properties window.

**Save a set of inspectors for later by using inspector groups.** Usually, many inspectors are shown at the same time. To use an inspector group, click ‘Inspector’ on the top menu of Designer. Here inspector group names are managed. The inspector group will save the inspector configuration, including positioning. To reposition an inspector group, simply adjust the inspectors and save the group again with the same name.

### Step Values and Fine Controls

Within the properties window, in the variables tab, many modules will have range attributes, with an optional ‘step’ to determine adjustable resolution for the slider or knob control. The Max and Min values determine the allowable tuning range, and can be adjusted at any time.

Fine controls of the tuning interface relies on a hotkey and click combination. _**Press and hold ctrl**_ **and scroll the mouse wheel on the knob/slider to use fine tuning.** For coarse tuning, hold shift and do the same. For adjusting with mouse clicks, the tuning is controlled by distancing the cursor away from the knob while tuning. This will allow for smaller changes to occur, as more distance is needed to create the angle.

### Viewing Module Variables on Canvas

A useful design feature is to see variable values without having to open all of the inspectors. The original design appears as:

![](../../.gitbook/assets/7%20%282%29.png)



**To show variable values, select “Module Variables” under the View menu.** With variables shown, it appears as:

![](../../.gitbook/assets/8.png)

### Module Status

Each module has an associated runtime status with 4 possible values:

_**Active**_ –The module's processing function is being called. This is the default behavior when a module is first instantiated.

_**Muted**_ – The module's processing function is not called. Instead, all of the output wires attached to the module are filled with zeros.

_**Bypassed**_ –The module's processing function is not called. Instead, the module's input wires are copied directly to its output wires. Some modules use an intelligent generic algorithm which attempts to match up input and output wires of the same size. Other modules implement custom bypass functions.

_**Inactive**_ –The module's processing function is not called and the output wire is untouched. This mode is used almost exclusively for debugging and the output wire is left in an indeterminate state. _**Use with caution!**_

Changing the module status is useful for debugging and making simple changes to the processing at run-time. The module status can be changed in both Design mode and Tuning mode.

**The Module Status can be changed by right-clicking on a module and selecting “Module Status** $$\to$$ **”.** To change the module status of a group, **select multiple modules\(including subsystems\) with drag and select or by pressing ctrl**, and right-click to change the status of all selected modules.

![](../../.gitbook/assets/9.png)



### Wires and Control Signals

Connections between audio modules are called _wires_ and correspond to buffers of data on the target. A wire has the following properties:

* number of channels
* block size
* sample rate
* complexity \(real or complex values\).

**Most modules can operate on an arbitrary number of channels, block size, and sample rate.** The number of channels and block size of a wire is called its _size._ Show wire properties using the View $$\to$$ Wire Types menu item.

#### Pin propagation

Wire information is resolved through a process known as _pin propagation._ The wire information is known from the system input and this information is propagated in module order until it reaches the system output pin. **To trigger pin propagation, click on the Propagate changes button \(** ![](../../.gitbook/assets/10.png) **\) on the toolbar.** Alternatively, right-click on an empty part of the canvas and select “Redraw” from the context menu. If there is a wiring error and pin propagation is unable to complete, an error sound will play. To find out more details to this error, run the system.

![](../../.gitbook/assets/13%20%281%29.png)

#### Feedback Wires

Feedback occurs when a wire is routed backwards to an input earlier in the system. Feedback wires must be manually specified. \[The manual specification of feedback wires is a limitation of the way that Audio Weaver resolves wiring information. Recall that Audio Weaver starts at the input of the system and then propagates wire size information module-by-module. When it first reaches a feedback point, it will have not yet resolved the size information for that wire and cannot proceed further. As a result, wires must be manually marked as feedback.\] **To make a feedback wire, right click a wire and select “Feedback”.** The wire will turn blue to indicate that it is a feedback wire. For each marked feedback wire, Audio Weaver will create a dedicated buffer to store its data. At system startup, data in the wire is initialized to all zeros. 

{% page-ref page="modules.md" %}

![](../../.gitbook/assets/11%20%283%29.png)

If a feedback wire is left _unmarked_, the following error will appear after an attempt to build the system:

![](../../.gitbook/assets/12%20%283%29.png)

To solve this, locate the feedback point in the block diagram and then mark the feedback wire.

**Wire size information must be set using the “Feedback Properties” dialog which is reached by right-clicking on the feedback wire.** The following dialogue box opens, allowing the ability to set the wire’s block size, number of channels, sample rate, data type, and whether or not it is complex. This should match the pins that the wire is connecting.

### Block Size

Each hardware target has a fundamental block size to specify how many samples per block are handled by the real-time audio I/O functions. This is shown on the Server window when the Server launches. \(See below.\)

Layouts can use any multiple of the target’s fundamental blockSize. **Change blockSize at any time by adjusting the hardware input parameters in SYS\_in**. BlockSize information is propagated from the input pin, through the modules, to the output pin. **The blockSize of the output pin is set by pin propagation**. \[The exact behavior is controlled by the checkbox “Validate system output pin” in Layout-&gt;Layout Properties dialog. There is the option of inheriting the output pin size from the wire attached to it \(Validate system output pin unchecked; this is the default\). Or force the wire to match the output pin \(Validate system output pin checked.\)\]

**Interpolator and decimator modules increase or decrease the blockSize**. Some modules also output a single value _control signal_ \(blockSize = 1\). For example, the BlockStatistics module can be configured to output the average value of a block of samples. This data will be one value per block. Control signals are drawn with dashed lines instead of solid lines.

![](../../.gitbook/assets/14%20%284%29.png)

In the example below, the `BlockStatistics1` module computes the RMS level of the signal and a lookup table \(TableInterp1\) turns this into a linear gain to be applied by Scaler1. The signals into and out of the lookup table module are control signals.

![](../../.gitbook/assets/15%20%283%29.png)

### Smoothed modules

Some modules have built in smoothing to prevent pops and clicks during tuning. Smoothing is implemented using a first order IIR filter:

𝑔𝑛=1−𝛼𝑔𝑛−1+𝛼𝑡\[𝑛\]

where 𝑔\[𝑛\] is the instantaneous smoothed gain and _t_\[𝑛\] is the discontinuous target gain. The coefficient controls the smoothing process and depends upon the _smoothingTime_ and the sampling rate. Smoothed modules take additional processing: only use these when tuning is needed during runtime.

The figure below shows a ScalerSmoothed module with a 10 msec time constant. The blue line is the instantaneous gain change from 1.0 to 2.0 and the red curve is the result of smoothing.

Time constants correspond to the familiar definition of time constants used in analog filters:

                                                           𝑔𝑡=𝑒−𝑡/𝜏

Where 𝜏 is the time constant. After 1 time constant has elapsed the gain has decayed by by $$e^{-1}=0.3679$$ . After 3 time constants, the gain has decayed by 95%, because 𝑒−3=0.0498. Thus, it takes several time constants for the gain change to fully take effect and this is reflected in the figure above.

## Modules in Browser Order

Audio Weaver has over 400 different types of modules available. This section organizes the modules into standard types with brief descriptions of each. Our focus will be on how to use the modules, with information provided in tables to determine the differences between similar modules.

### Annotation

In order to keep notes within layouts, Designer supports text boxes, rectangle panels, and arrows. While there are many ways to use these, the standard is to break the layout file into “processing sections” with information on how to tune the design. Annotation is also good for keeping “presets” or “modes” written down on the canvas.

![](../../.gitbook/assets/image%20%285%29.png)

#### Documenting Layouts

To use the annotations, drag them onto the browser. Rescale them and position them accordingly. To edit a text box, double click it and type away on the canvas. To change text size or annotation color/width, check the properties panel for each annotation. In order to keep a standard across the design file, it is recommended to establish a standard annotation \(i.e. get the fonts/size/color set up\) and copy/paste this to keep the style throughout the annotations.  

![](../../.gitbook/assets/image%20%281%29.jpeg)

### Delays

Delay modules hold the input signal for some amount of time using an internal circular buffer. This buffer is instantiated with a size _maxDelay_. At runtime, _currentDelay_ is set to control the time constant for the maxDelay buffer. Most delays update currentDelay instantaneously, but smoothed\(interpolated\) delays are provided if the final product needs a varying delay. 

Delay time type can be int or fract32 samples, int or fract32 milliseconds, or blocks. Input type also varies, meaning some modules take Float audio data, some take Fract32 audio, and some take any type \(including int\). 

For most cases, use Delay msec or Delay samples. Modulated delays come into play for making musical effects.

#### Allpass Delays

Allpass Delays use feedback and feedforward in order to vary the phase of a signal without changing its magnitude or its position in time. Our allpass delays have a coef variable which determines the amount of gain on the feedback/forward mix factor. Allpasses are commonly used in audio effects like chorus, flanger, reverb, and stereo effects. Our allpasses also have the option of outputting the delayed signal \(in cases where time position and phase should change\). 

#### Modulated Delays

Modulated Delays come with a modulation control pin. This control pin feeds the modulation factor _mod_, limited in depth by _modDepth_. A common use case is to provide the mod pin with a random or oscillator source that varies from -1 to 1, allowing the delay to vary. This oscillation reduces harmonic artifacts from feedback delay lines.

#### Delay Taps

Delay Taps create an evenly spaced N amount of delays, which are all interleaved into separate ‘delay channels.’ This is useful for adaptive FIR filters and prediction algorithms due to the consistency in delay length.

#### Low Memory Delays

Since delays tend to be memory intensive, memory efficient options are provided. Modules labelled with a 16 in the name use half of the bits \(compared to 32 bit data\). This results in about half the size on the heap, at a cost of lost amplitude resolution. Our most memory efficient delay for multiple lines is the Delay State Writer16. This uses a circular buffer and multiple pointers instead of multiple buffers in memory, all while using 16-bit resolution.

#### Table of Delay Modules

![](../../.gitbook/assets/image%20%283%29.png)

### DSP Concepts IP

#### Long FIR Filter

This **zero latency FIR** allows for large FIR filters by breaking a large convolution into multiple smaller convolutions. This module supports one channel of audio. **The larger the blockSize, the more efficient the processing**. It expects time domain FIR coefficients, and will automatically convert these into the frequency domain for processing. It is recommended to load the coefficients as a text file using the properties browser to edit the FIR array, rather than typing them in by hand. 

####  WOLA Forward Filterbank \(Analysis\)

![](../../.gitbook/assets/image%20%2838%29.png)

This module is used to convert a time domain\(real number\) signal into frequency domain\(complex number\) bins. The output of this module will operate according to this blocksize. From this point on, the data is complex. See the Frequency Domain folder for modules that operate within the complex domain. The first and last bin represent DC, and have no complex data.

#### WOLA Inverse Filterbank \(Synthesis\)

![](../../.gitbook/assets/image%20%2842%29.png)

This module will convert a block of frequency bin\(complex number\) data into the time domain\(real number\). This is normally paired with WOLA Forward Filterbank \(Analysis\). Be sure to keep the WOLA Forward and Inverse sizes the same. More information is provided in the Frequency Domain section.

### Dynamics

Dynamic modules de/emphasize the amplitude structure of data. AGC stands for Automatic Gain Control. It differs from standard gain modules in that the gain value changes by itself over time, and the gain can scale loud volumes separate from quiet volumes. This ‘warping’ of the volume information can regulate perceived volume, maximize output, add a safety stage before the signal hits the speakers, remove low volume noise, and manage sound source priority like ducking music behind a dialog track. This section of the guide will cover the modules in the AGC folder: compressor core, envelope followers, limiter core, expander\(and noise gate\), ducker, and agc core. It will then go over building a custom AGC, and finally optimize an AGC to run at reduced computation cycles.

  
The dynamics modules operate with different gain variation speeds and different magnitude reference signals. **All dynamics modules with the suffix \*\*\*Core do not generate audio data, but instead output volume data to be sent to a scaler control pin or an AGC Multiplier.** The following diagrams represent the typical input/output response for various AGC types. 

![](../../.gitbook/assets/image%20%289%29.jpeg)

_**Compressor**_ – reduce the peak dynamic range of a signal

![](../../.gitbook/assets/image%20%285%29.jpeg)

_**Downward expander \(and noise gate\)**_ – reduce small signal levels and behave as noise gates

![](../../.gitbook/assets/image.jpeg)

_**Limiter**_ – restrict peak signal levels to avoid digital clipping _\(limiters generally have a horizontal or flat slope, which means high ratio\)_

![](../../.gitbook/assets/image%20%2839%29.png)

_**Ducker**_ ****– use a trigger signal to determine when to boost or reduce gain of another signal.

![](../../.gitbook/assets/image%20%2816%29.jpeg)

_**AGC Core**_– adjust the gain to keep the signal within a specified RMS range

#### Compressors

A compressor reduces the signal’s dynamic range, meaning that it lowers the level of loud signals and boosts quiet ones, reducing the difference between loud and soft signals. Make-up gain is usually applied after for increasing the perceived loudness. This can be used for scenarios when keeping the overall volume low is desired but hearing small details is still important, such as night-time movie watching. The behavior of the compressor is best understood by looking at its input-output response:

![](../../.gitbook/assets/image%20%2812%29.jpeg)

Above the threshold the compressor reduces the signal level; below the threshold the compressor increases the signal level \(it’s expanding the system\). This brings all output signals closer to the threshold level and reduces the overall dynamic range. The AGCCompressorCore module is wired the same way as the limiter module, receiving its input from an Abs or MaxAbs module and outputting to a multiplier, as shown below:

![](../../.gitbook/assets/image%20%287%29.jpeg)

#### Envelope Modulation

Envelope modulators control the impact that relative peaks have. The Attack Release module uses attackTimeInitial and attackTimeFinal to smooth peaks. The Attack Decay Sustain Release is similar, but also includes 2 stages in between the attack and release. Decay lowers the level into a hold that is based on the sustain level. After this hold ends, the release occurs.

![](../../.gitbook/assets/image%20%2830%29.png)

#### Limiters

The AGCLimiterCore module is parameterized by its threshold, ratio, gain, knee depth, attack time, and decay time. The AGCLimiterCore module computes the time varying gain.

![](../../.gitbook/assets/image%20%2817%29.jpeg)

Take the absolute value of the signal so that the AGCLimiterCore module treats positive and negative signals equally. The first pin on the AGCMultiplier is the gain to apply and the second input is the audio signal itself.

![\(standard arrangement for a mono limiter\)](../../.gitbook/assets/image%20%288%29.jpeg)

When the input is below the threshold the line has a slope of 1 indicating that the signal level is unchanged. Above the threshold the slope drops indicating that the output level will be reduced compared to the input level. Limiters have a parameter called the “ratio” referring to the reciprocal slope of the gain ratio **above the threshold**. A high ratio provides hard limiting, close to 1 provides gentle limiting.

The limiter applies a piecewise function to determine its gain: at a specified threshold, the slope changes. The transition between sections is smoothed by a connecting polynomial section, often referred to as the “knee”. The knee provides a gentle polynomial interpolation between the threshold and the requested slope. The kneeDepth parameter controls the extent of the polynomial section. The polynomial starts at threshold-kneeDepth and ends at threshold+kneeDepth.

![](../../.gitbook/assets/image%20%283%29.jpeg)

The speed with which a limiter responds to an increase in sound level is described by its “attack time”. The lower the attack time, the faster the limiter will respond to the sound level rising above its threshold. Similarly, decay \(or release\) time describes the speed with which the limiter’s effect is relaxed after the sound level drops back under its threshold. The time behavior of the dynamics processors is implemented with first order IIR smoothers with different attack and decay coefficients. The above image shows example attack and decay curves.

#### Downward Expander

The DownwardExpanderCore module is also a limiter with a piecewise gain, but its piecewise function is different. Whereas most limiters use a slope of 1 below a threshold and a reduced slope above it, this module features a very steep slope below its threshold and a slope of 1 above it. Rather than reducing the level of loud signals, this reduces the level of quiet signals. The DownwardExpanderCore module’s response behavior is shown below:

![](../../.gitbook/assets/image%20%2840%29.png)

One use of this module is for filtering out low-level noise while retaining a louder signal. This is very useful for eliminating “hiss,” low level background noise in a signal. Like most dynamic processing modules, the DownwardExpanderCore is designed to take its input from the MaxAbs module and output its gain as an input to the AGCMultiplier module. Shown below is an example of a noise gate which eliminates low level signals.

![](../../.gitbook/assets/image%20%286%29.jpeg)

#### AGC Core

![](../../.gitbook/assets/image%20%2814%29.jpeg)

![](../../.gitbook/assets/image%20%2827%29.png)

This module has a slowly varying volume control which transfers the **input signal** level towards a targetLevel, a specified RMS level. The input RMS is smoothed via the smoothing time variable. This allows the gain to change gradually. The gain is limited to the range \[-maxAttenuation and maxGain\]. The ratio control determines the speed of the gain change for all signals above the **activation Threshold**. When the level of the input signal falls below **activationThreshold**, the AGCCore holds the last gain setting. If the enableRecovery checkbox is checked, the gain will slowly return to 0 dB when not activated. The rate of return is governed by recoveryRate.

















