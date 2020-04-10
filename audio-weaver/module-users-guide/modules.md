# Modules

Audio Weaver has over 400 different types of modules available. This section organizes the modules into standard types with brief descriptions of each. Our focus will be on how to use the modules, with information provided in tables to determine the differences between similar modules.

### Annotation

In order to keep notes within layouts, Designer supports text boxes, rectangle panels, and arrows. While there are many ways to use these, the standard is to break the layout file into “processing sections” with information on how to tune the design. Annotation is also good for keeping “presets” or “modes” written down on the canvas.

![](../../.gitbook/assets/16.png)

#### Documenting Layouts

To use the annotations, drag them onto the browser. Rescale them and position them accordingly. To edit a text box, double click it and type away on the canvas. To change text size or annotation color/width, check the properties panel for each annotation. In order to keep a standard across the design file, it is recommended to establish a standard annotation \(i.e. get the fonts/size/color set up\) and copy/paste this to keep the style throughout the annotations.

![](../../.gitbook/assets/17%20%283%29.png)

### Delays

Delay modules hold the input signal for some amount of time using an internal circular buffer. This buffer is instantiated with a size `maxDelay`. At runtime, `currentDelay` is set to control the time constant for the `maxDelay` buffer. Most delays update `currentDelay` instantaneously, but smoothed\(interpolated\) delays are provided if the final product needs a varying delay.

Delay time type can be int or fract32 samples, int or fract32 milliseconds, or blocks. Input type also varies, meaning some modules take Float audio data, some take Fract32 audio, and some take any type \(including int\).

For most cases, use Delay msec or Delay samples. Modulated delays come into play for making musical effects.

#### Allpass Delays

Allpass Delays use feedback and feedforward in order to vary the phase of a signal without changing its magnitude or its position in time. Our allpass delays have a coef variable which determines the amount of gain on the feedback/forward mix factor. Allpasses are commonly used in audio effects like chorus, flanger, reverb, and stereo effects. Our allpasses also have the option of outputting the delayed signal \(in cases where time position and phase should change\).

#### Modulated Delays

Modulated Delays come with a modulation control pin. This control pin feeds the modulation factor _mod_, limited in depth by _modDepth_. A common use case is to provide the _mod_ pin with a random or oscillator source that varies from -1 to 1, allowing the delay to vary. This oscillation reduces harmonic artifacts from feedback delay lines.

#### Delay Taps

Delay Taps create an evenly spaced N amount of delays, which are all interleaved into separate ‘delay channels.’ This is useful for adaptive FIR filters and prediction algorithms due to the consistency in delay length.

#### Low memory delays

Since delays tend to be memory intensive, memory efficient options are provided. Modules labelled with a 16 in the name use half of the bits \(compared to 32 bit data\). This results in about half the size on the heap, at a cost of lost amplitude resolution. Our most memory efficient delay for multiple lines is the Delay State Writer16. This uses a circular buffer and multiple pointers instead of multiple buffers in memory, all while using 16-bit resolution.

#### Table of Delay Modules

{% embed url="https://docs.google.com/spreadsheets/d/1VgcCQeckliYN1\_l1bZqXrYjrTRq3-c\_eEpKU\_NOIExM/edit?usp=sharing" %}



### DSP Concepts IP

#### Long FIR Filter

This **zero latency FIR** allows for large FIR filters by breaking a large convolution into multiple smaller convolutions. This module supports one channel of audio. **The larger the blockSize, the more efficient the processing**. It expects time domain FIR coefficients, and will automatically convert these into the frequency domain for processing. It is recommended to load the coefficients as a text file using the properties browser to edit the FIR array, rather than typing them in by hand.

#### WOLA Forward Filterbank \(Analysis\)

This module is used to convert a time domain\(real number\) signal into frequency domain\(complex number\) bins. The output of this module will operate according to this blocksize. From this point on, the data is complex. See the Frequency Domain folder for modules that operate within the complex domain. The first and last bin represent DC, and have no complex data.

![](../../.gitbook/assets/19%20%282%29.png)

#### WOLA Inverse Filterbank \(Synthesis\)

This module will convert a block of frequency bin\(complex number\) data into the time domain\(real number\). This is normally paired with WOLA Forward Filterbank \(Analysis\). Be sure to keep the WOLA Forward and Inverse sizes the same. More information is provided in the Frequency Domain section.

![](../../.gitbook/assets/20%20%281%29.png)

### Dynamics

Dynamic modules de/emphasize the amplitude structure of data. AGC stands for Automatic Gain Control. It differs from standard gain modules in that the gain value changes by itself over time, and the gain can scale loud volumes separate from quiet volumes. This ‘warping’ of the volume information can regulate perceived volume, maximize output, add a safety stage before the signal hits the speakers, remove low volume noise, and manage sound source priority like ducking music behind a dialog track. This section of the guide will cover the modules in the AGC folder: compressor core, envelope followers, limiter core, expander\(and noise gate\), ducker, and agc core. It will then go over building a custom AGC, and finally optimize an AGC to run at reduced computation cycles.

The dynamics modules operate with different gain variation speeds and different magnitude reference signals. **All dynamics modules with the suffix \*\*\*Core do not generate audio data, but instead output volume data to be sent to a scaler control pin or an AGC Multiplier.** The following diagrams represent the typical input/output response for various AGC types.

_**Compressor**_ – reduce the peak dynamic range of a signal

![](../../.gitbook/assets/21.png)

_**Downward expander \(and noise gate\)**_ – reduce small signal levels and behave as noise gates



![](../../.gitbook/assets/22%20%283%29.png)

_**Limiter**_ – restrict peak signal levels to avoid digital clipping \(_limiters generally have a horizontal or flat slope, which means high ratio_\)

![](../../.gitbook/assets/23%20%283%29.png)

_**Ducker**_ **-** use a trigger signal to determine when to boost or reduce gain of another signal.

![](../../.gitbook/assets/24%20%281%29.png)

_**AGC**_ ****_**Core**_– adjust the gain to keep the signal within a specified RMS range



![](../../.gitbook/assets/25%20%281%29.png)

#### Compressors

A compressor reduces the signal’s dynamic range, meaning that it lowers the level of loud signals and boosts quiet ones, reducing the difference between loud and soft signals. Make-up gain is usually applied after for increasing the perceived loudness. This can be used for scenarios when keeping the overall volume low is desired but hearing small details is still important, such as night-time movie watching. The behavior of the compressor is best understood by looking at its input-output response:

![](../../.gitbook/assets/26.png)

Above the threshold the compressor reduces the signal level; below the threshold the compressor increases the signal level \(it’s expanding the system\). This brings all output signals closer to the threshold level and reduces the overall dynamic range. The AGCCompressorCore module is wired the same way as the limiter module, receiving its input from an Abs or MaxAbs module and outputting to a multiplier, as shown below:

![](../../.gitbook/assets/27%20%282%29.png)

#### Envelope Modulation

![](../../.gitbook/assets/28%20%281%29.png)

Envelope modulators control the impact that relative peaks have. The Attack Release module uses attackTimeInitial and attackTimeFinal to smooth peaks. The Attack Decay Sustain Release is similar, but also includes 2 stages in between the attack and release. Decay lowers the level into a hold that is based on the sustain level. After this hold ends, the release occurs.

#### Limiters

The AGCLimiterCore module is parameterized by its threshold, ratio, gain, knee depth, attack time, and decay time. The AGCLimiterCore module computes the time varying gain.

![](../../.gitbook/assets/29%20%283%29.png)

Take the absolute value of the signal so that the AGCLimiterCore module treats positive and negative signals equally. The first pin on the AGCMultiplier is the gain to apply and the second input is the audio signal itself.

![\(standard arrangement for a mono limiter\)](../../.gitbook/assets/30%20%282%29.png)

When the input is below the threshold the line has a slope of 1 indicating that the signal level is unchanged. Above the threshold the slope drops indicating that the output level will be reduced compared to the input level. Limiters have a parameter called the “ratio” referring to the reciprocal slope of the gain ratio **above the threshold**. A high ratio provides hard limiting, close to 1 provides gentle limiting.

The limiter applies a piecewise function to determine its gain: at a specified threshold, the slope changes. The transition between sections is smoothed by a connecting polynomial section, often referred to as the “knee”. The knee provides a gentle polynomial interpolation between the threshold and the requested slope. The kneeDepth parameter controls the extent of the polynomial section. The polynomial starts at threshold-kneeDepth and ends at threshold+kneeDepth.

![](../../.gitbook/assets/31%20%282%29.png)

The speed with which a limiter responds to an increase in sound level is described by its “attack time”. The lower the attack time, the faster the limiter will respond to the sound level rising above its threshold. Similarly, decay \(or release\) time describes the speed with which the limiter’s effect is relaxed after the sound level drops back under its threshold. The time behavior of the dynamics processors is implemented with first order IIR smoothers with different attack and decay coefficients. The above image shows example attack and decay curves.

#### Downward Expander

The DownwardExpanderCore module is also a limiter with a piecewise gain, but its piecewise function is different. Whereas most limiters use a slope of 1 below a threshold and a reduced slope above it, this module features a very steep slope below its threshold and a slope of 1 above it. Rather than reducing the level of loud signals, this reduces the level of quiet signals. 

![The DownwardExpanderCore module&#x2019;s response behavior](../../.gitbook/assets/31%20%282%29.png)

One use of this module is for filtering out low-level noise while retaining a louder signal.This is very useful for eliminating “hiss,” low level background noise in a signal. Like most dynamic processing modules, the DownwardExpanderCore is designed to take its input from the MaxAbs module and output its gain as an input to the AGCMultiplier module. 

![Example of a noise gate which eliminates low level signals](../../.gitbook/assets/33%20%283%29.png)

#### 

#### AGC Core

This module has a slowly varying volume control which transfers the **input signal** level towards a targetLevel, a specified RMS level. The input RMS is smoothed via the smoothing time variable. This allows the gain to change gradually. The gain is limited to the range \[-maxAttenuation and maxGain\]. The ratio control determines the speed of the gain change for all signals above the **activation Threshold**. When the level of the input signal falls below **activationThreshold**, the AGCCore holds the last gain setting. If the enableRecovery checkbox is checked, the gain will slowly return to 0 dB when not activated. The rate of return is governed by recoveryRate.

#### ![](../../.gitbook/assets/35%20%282%29.png)

#### ![](../../.gitbook/assets/34%20%281%29.png)

#### Table of Dynamics Modules

{% embed url="https://docs.google.com/spreadsheets/d/1w9kcz2lJpEbb3-4QUFFJOLjd3mG8YdyQtbz3xNvFQ0A/edit?usp=sharing" %}



### Filters

 The Audio Weaver Filters folder lists over 60 filters. They have been broken down according to user needs, with the folder labels Adaptive, Calculated Coeffs, Controllable, High Precision, Raw Coeffs, and list the most commonly used filters. The Adaptive folder contains the LMS module, an adaptive filter with tracking capabilities. For those users less experienced with designing filters, the Calculated Coeffs filters take in frequency information, Q, Gain, and type, similar to tuning a filter in a DAW. Users with more DSP background can use the Raw Coeffs filters to tune filters with mathematical information. The most frequently used filters are the ButterworthFilter \(highpass, lowpass, allpass\), SecondOrderFilterSmoothed, with 20 different filter types, and the SecondOrderFilterSmoothedCascade: multiple 2nd order filters in series.

#### Adaptive \(LMS\)

The LMS filter predicts the FIR of a system whose transfer function is not given. It’s input and output adapt or “predict” what the system response is. Filter weights are updated over time based on mu speed, higher numbers being the faster update speed. Higher numtaps give higher chance to converge with the optimum filter weight\(meaning less error\). The error can be tracked realtime with the errorSignal output. The module comes with an option to output the predicted “coeffs”. The following system shows white noise being ran through a 10 point FIR. The LMS will predict the FIR coefficients, and sinks will display the error and coeff function.

![](../../.gitbook/assets/37%20%282%29.png)

 This sink shows the coeff prediction.

![](../../.gitbook/assets/38.png)



The LMS is trying to predict this FIR response.

The Error2 display shows a value of -125 dB, which means that our signal is very accurate. The sink to the right displays this as well.

#### Filters with Calculated Coeffs

Audio Weaver has a several filters with built-in design equations. These filters are implemented using Biquad or BiquadCascade modules behind the scenes and the filter coefficients are computed by the design equations based on high-level filter specifications. The design equations use the sample rate on the input wire when computing coefficients. The following sections describe each of the calculated coeffs modules.

**Allpass Pair**

The Allpass Pair module creates a pair of allpass filters with the special property that their sum and difference form a doubly complementary highpass-lowpass filter pair. When used in conjunction with the Sum and Difference module \(available in the Math folder\) this Allpass Pair module can be used to construct more complex structures such as N-way crossovers and filter banks. The following design mixes one channel of white noise into two “bands” of white noise using this technique. The sink is an FFT showing the frequency response of the signal.

![](../../.gitbook/assets/39%20%282%29.png)

**Audio Weighting Filters**

The AudioWeighting module is located under Filters/Calculated Coeffs This module applies different standard audio weighting to a signal. The available weightings are selectable from the inspector and include: A, B, C, and D-weighting, as well as ITU468, LeqM, and ITU 1770. The WeightingFilter is used for noise measurements and broadcast loudness applications. The following frequency responses display each weighting’s filter.

![](../../.gitbook/assets/40%20%283%29.png)![](../../.gitbook/assets/41%20%283%29.png)![](../../.gitbook/assets/42.png)

![](../../.gitbook/assets/43%20%282%29.png)![](../../.gitbook/assets/44.png)![](../../.gitbook/assets/45.png)

![](../../.gitbook/assets/46.png)

**Crossover Filter**

A crossover is a special type of filter that splits a signal into multiple bands, while sustaining a total gain of 0 dB. It will boost the level of one frequency band as the other drops to compensate to 0 dB gain. This behavior is shown in the figure below.

![](../../.gitbook/assets/47%20%281%29.png)

Crossovers are used in loudspeaker applications to separate signals into different frequency bands to be output via woofers, mid-range, and tweeter speakers. They are implemented using ButterworthFilter \(odd-order\) or Linkwtiz-Riley \(even-order\) filters. Crossover filters can be made manually using individual filters. By cascading filters and applying an allpass filter during other lane filter stages \(shown below\), more crossover points can be added and the signal split into more frequency bands, while retaining the unity gain property.

Alternatively, use the crossover module which contains all of the needed filters. The Crossover Filter module allows the user to set the type of filter, number of output bands, and filter order, specified in the module’s properties. Crossovers are used for separating different frequency bands of a signal. The example below demonstrates a crossover module being used to split a signal into a high band above 250 Hz and a low band below 250 Hz. The sum of the levels of the two bands is always 0 dB. As the input frequency changes near 250 Hz, one channel’s level drops and the other smoothly increases to compensate to 0 dB.

![](../../.gitbook/assets/48%20%281%29.png)

![](../../.gitbook/assets/49%20%282%29.png)









![](../../.gitbook/assets/51.png) 

This is the same behavior as using two ButterworthFilter filters:

![](../../.gitbook/assets/50%20%282%29.png)![](../../.gitbook/assets/52%20%281%29.png)



The Crossover Filter module allows the configuration of 2 or more output channels. The module will then be drawn with 3 output pins. The top pin is the low frequency; the center pin is the mid-range, and the bottom pin is the high frequencies. The inspector shows two cutoff frequencies: between the low and mid-range; and between the mid-range and high frequencies. For example, if implementing a 3-way loudspeaker crossover, configure it as:

![](../../.gitbook/assets/54%20%281%29.png)

\*\*\*\*

**Emphasis Filter**

The EmphasisFilter module implements a pre-emphasis or de-emphasis, used for noise reduction. The cutoff frequency is specified by the time constant tau, which is set in the inspector. The examples below show emphasis and de-emphasis filters with 75 microsecond time contants.

![](../../.gitbook/assets/55%20%281%29.png) ![](../../.gitbook/assets/56%20%282%29.png)

![](../../.gitbook/assets/53%20%282%29.png)

#### **Graphic EQs**

The GraphicEQ module splits up a signal into different bands and independently attenuates or amplifies each band. Under module names and arguments, the number of bands and the order of each filter are set. The bands are logarithmically spaced across the Nyquist frequency of the input signal. Each band’s gain can be set in the inspector. This EQ can automatically adjust its bands based on the lowEdge and highEdge arguments. After setting this to the desirable range, change the resetCenterFreqs flag to ‘1’. This will recalculate the bands, throwing away all slider data \(so change the bands before tuning\).

{% hint style="info" %}
**The more bands there are, the higher the filter order should be to better isolate the bands.**
{% endhint %}

The GraphicEQBand module applies a gain to a specific frequency band. This module is the building block of the GraphicEQ. In the inspector, the gain, lower edge frequency and upper edge frequency are specified. The filter order is specified from by the module properties. This module is not typically used directly; use GraphicEQ instead.

**Hilbert**

The Hilbert module can be considered a filter which simply shifts phases of all frequency components of its input by -n/2 radians. This operates on complex \(real and imaginary\) data input and output.

**Pink Filter**

The PinkFilter is a low pass filter with a -3dB/octave slope. It is used to generate pink noise from white noise.

**Three Band Tone Control**

The ThreeBandToneControl module is similar to the GraphicEQ except there are only 3 bands. The low, mid, and high bands’ middle frequency and gain are set in the inspector. The ThreeBandToneControl module is very efficient and uses first order shelf filters for the low and high frequency gain adjustments. The middle band is a simple gain and the net result is that the ThreeBandToneControl takes as much computation as a single BiquadSmoothed filter.

#### Controllable Filters

The filters presented thus far get their high level design parameters from the inspectors. At times, it is useful to have a filter whose parameters are controlled by other signals or modules in the layout, such as a source or a hardware input pin. This is called a _controllable filter_. The controllable filters folder includes a first and second order filter, along with a lowpass filter, which all have control pins for their frequency. The second order filter can also enable more pins, like Q and Gain depending on the module arguments.

**First Order Filter Control**

The FOFControl module implements a first order lowpass or highpass filter. The control pin specifies the cutoff frequency of the filter and the design equations are executed every block allowing very rapid updates.

**LPF Control**

![](../../.gitbook/assets/57%20%282%29.png)![](../../.gitbook/assets/58%20%283%29.png)

The LPF Control module is a time varying first order low pass with smoothly varying frequency based on the input pin.

**Second Order Filter Control**

The SOFControl filter is built upon the SecondOrderFilterSmoothed module. It has a fixed filterType which is specified on the inspector. Under module arguments, specify which of the filter design parameters should be obtained via input pins:

![](../../.gitbook/assets/59.png)

Parameters which aren’t specified by input pins are specified via the variables and properties tab. The SOFControl module uses deferred processing to compute the filter coefficients. That is, the design equation is not executed every block. Rather, when the control data on the input pin changes, the module sets a bit in its instance structure which causes the design function to be called from non-real-time code. This reduces the peak CPU load at the expense of having a slower update rate. In typical applications, the module will update every few 10s of milliseconds. If updating needs to happen more quickly, then use the FOFControl module or the ParamSet module coupled with a SecondOrderFilterSmoothed module.

A very common use of the SOFControl module is within a perceptual volume control. As the volume of the system is reduced, overall spectral balance should be maintained. Due to the sensitivity of the human auditory system, low frequencies and high frequencies appear to drop off more quickly than mid frequencies. Thus, to maintain the overall spectral balance, boost low and high frequencies as the volume level decreases. The VolumeControl module accomplishes this with a fixed boost table. For finer control over the boost, use a TableInterp module together with a SOFControl filter as shown below. The control signal “Volume” specifies the listening level and ranges from 0 \(loud\) to -80 \(soft\). The lookup tables convert the Volume setting into low frequency and high frequency boosts which are applied using the SOFControl module. The low frequency SOFControl module implements a peaking filter at 40 Hz and the gain is taken from the control pin. The high frequency SOFControl module implements a high shelf in which the gain is taken from the control pin.

![Example showing use of the SOFControl module in a table driven loudness control](../../.gitbook/assets/60%20%283%29.png)

![](../../.gitbook/assets/61%20%283%29.png)![](../../.gitbook/assets/62%20%282%29.png)

_Figure 2. Lookup tables used in the table driven loudness control example. The left table converts the volume setting into a bass boost and the right table does the same for the high frequency adjustment._

#### Filters with Raw Coeffs

Audio Weaver contains several filter types which operate on raw coefficients. These filters are for expert users who understand DSP and know how to calculate the filter coefficients[\[3\]](). There are two types of filters – Finite Impulse Response \(FIR\) and Infinite Impulse Response \(IIR\). Although Audio Weaver supports both types of filters, the majority of the filters used in audio applications are IIR due to their computational efficiency.

The most basic IIR filter is the Biquad and it is implemented with the difference equation:

𝑎0𝑦𝑛=𝑏0𝑥𝑛+𝑏1𝑥𝑛−1+𝑏2𝑥𝑛−2−𝑎1𝑦𝑛−1−𝑎2𝑦𝑛−2

There are 5 coefficients that the user must set: b0, b1, b2, a1, and a2 \(a0 is always assumed to be 1\). Audio Weaver does not check for stability and care must be used when computing the filter coefficients. There are several variants of Biquad filters. The simples – Biquad – has a single stage and implements the different equation shown above. BiquadCascade implements N stages of filtering with each channel using the same coefficients. BiquadNCascade implements N stages with each channel have its own set of coefficients. Finally, BiquadSmoothed implements a single Biquad stage with coefficient smoothing on a block-by-block basis.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Icon</th>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/63 (1).png" alt/>
      </td>
      <td style="text-align:left">FIR</td>
      <td style="text-align:left">
        <p>Time domain FIR filter</p>
        <p>Specify filter length in module properties</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/64 (1).png" alt/>
      </td>
      <td style="text-align:left">Biquad</td>
      <td style="text-align:left">
        <p>Second order IIR filter.</p>
        <p>5 filter coefficients are specified.</p>
        <p>No smoothing.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/65 (2).png" alt/>
      </td>
      <td style="text-align:left">BiquadCascade</td>
      <td style="text-align:left">
        <p>Multiple Biquad filters in series.</p>
        <p>The number of filters is specified in module properties.</p>
        <p>The same coefficients are used per channel.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/66.png" alt/>
      </td>
      <td style="text-align:left">BiquadSmoothed</td>
      <td style="text-align:left">
        <p>Second order IIR filter.</p>
        <p>5 filter coefficients are specified.</p>
        <p>Smoothed on a block-by-block basis</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/67 (2).png" alt/>
      </td>
      <td style="text-align:left">BiquadNCascade</td>
      <td style="text-align:left">
        <p>Multiple Biquad filters in series.</p>
        <p>The number of filters is specified in module properties.</p>
        <p>Different coefficients are used per channel.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/68 (1).png" alt/>
      </td>
      <td style="text-align:left">FIR Sparse</td>
      <td style="text-align:left">
        <p>Sparse FIR filter in which most values are zero.</p>
        <p>Less convolution cycles than normal FIR</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/69 (2).png" alt/>
      </td>
      <td style="text-align:left">FIR Sparse Reader</td>
      <td style="text-align:left">
        <p>Sparse FIR that connects to a delay state writer.</p>
        <p>Convolution is based on a pointer rather than a separate FIR buffer.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/fir-sparse-reader-fract-16.png" alt/>
      </td>
      <td style="text-align:left">FIR Sparse Reader Fract16</td>
      <td style="text-align:left">
        <p>Like FIR Sparse Reader except half the memory.</p>
        <p>Data is converted to fract16 for computations and has a conversion for
          the output if necessary.</p>
      </td>
    </tr>
  </tbody>
</table>#### High Precision Filters

Audio Weaver contains a variety of Biquad filters for equalizing audio. Some filters require raw coefficients \(such as Biquad or BiquadCascade\) while others contain built-in design equals \(such as the SecondOrderFilter or ButterworthFilter\). These filters are implemented using a Direct Form 2 \(DF2\) structure:

 

![](../../.gitbook/assets/screen-shot-2020-03-11-at-10.35.25-pm.png)

All Biquad filters including the DF2 have 5 coefficients. The advantage of the DF2 structure is that it requires only 2 state variables per filter as compared to 4 state variables for the DF1 structure.

These Biquad filters are implemented using floating-point arithmetic and are generally fine for most audio applications. Floating-point arithmetic, though, is not a panacea for all numerical issues and these filters can still suffer from quantization noise. The noise manifests itself as low-level noise correlated with the level of the input signal. Quantization noise is exacerbated by high sampling rates \(96 kHz and above\) and by having poles very close to the unit circle and this usually arises when making very low frequency EQ changes.

To solve these noise issues Audio Weaver includes a High Precision filter modules. These modules use floating-point input and output data and are compatible with the other floating-point modules. Internally the high precision filters use a proprietary DSP Concepts filter structure which significantly reduces quantization noise. The filters are also efficient with a typical Biquad requiring 7 MAC operations vs the 5 needed in a DF2 Biquad.

The High Precision modules are designed to be drop in replacements for the non-high precision filters. That way, numerical problems can be resolved by replacing the offending filter with its high precision version.

| Icon | Name | Description |
| :--- | :--- | :--- |
| ![](../../.gitbook/assets/72.png) | BiqudSmoothedHP | Smoothly varying Biquad |
| ![](../../.gitbook/assets/73%20%281%29.png) | ButterworthFilterHP | Butterworth lowpass, highpass, and allpass filters |
| ![](../../.gitbook/assets/74.png) | BiquadCascadeHP | Cascade of N Biquad stages |
| ![](../../.gitbook/assets/75%20%281%29.png) | GraphicEQBandHP | Single band of a graphic equalizer |
| ![](../../.gitbook/assets/76%20%281%29.png) | SOFControlHP | Controllable second order filter with design equations |
| ![](../../.gitbook/assets/77.png) | SOFCascadeHP | Cascade of second order filters each with design equations |
| ![](../../.gitbook/assets/78.png) | SecondOrderFilterHP | Single second order filter with design equations |
| ![](../../.gitbook/assets/79.png) | VolumeControlHP | Fletcher Munson volume control with loudness compensation |

The crossover filter module \(XoverNway\) is actually a subsystem consisting of multiple individual modules. The module properties give the option to construct the crossover using standard Biquads or high precision Biquads:

![](../../.gitbook/assets/80%20%281%29.png)

The graphic equalizer gives the option of using standard precision or high precision filters.

![](../../.gitbook/assets/81.png)Here is an example of the benefits of the high precision filter. The system in the example has a peaking filter at 20 Hz with a gain of 6 dB and a Q of 2 and operates at a 48 kHz sample rate. The total harmonic distortion and noise \(THD+N\) for different input frequencies is plotted below. First for standard Biquad filters



![](../../.gitbook/assets/picture1.png)

![](../../.gitbook/assets/82%20%281%29.png)And now with a high precision filter, notice that the noise floor is reduced significantly – by up to 90 dB at low frequencies.

![](../../.gitbook/assets/picture2.png) 

For the interested reader, this measurement is performed by passing sine waves of different frequencies through the filter. Apply a notch filter at the output which removes the sine wave and then measure the RMS energy in the residual. This residual energy equals the THD+N. The measurement is repeated for many different frequencies and the plot reflects the measured THD+N at each input frequency.

#### Common Filter Modules

The following filters are found as modules with no folder in the Filters directory. This is because they are the most common types of filters, which cover most general cases of filtering needs.

**ButterworthFilter**

This module implements lowpass, highpass, or allpass filters using a Butterworth design. The filters have a gain of 0 dB in the passband and are then monotonically decreasing in the stopband. The filter order is specified under module properties and ranges from 1st order \(6dB/octave\) to 10th order \(60dB/octave\). The filter order can only be changed in Design mode. Specify the filter type on the inspector \(lowpass, highpass, or allpass\) as well as the cutoff frequency, in Hz. Since these parameters are on the inspector, the filter type and cutoff frequency can be changed at run-time. Unfortunately, the ButterworthFilter does not have coefficient smoothing and there may be discontinuities when coefficients are updated.

![Butterworth lowpass filter frequency response as a function of filter order. The filter order goes from 1st order \(least steep line\) to 10th order \(steepest line\). The cutoff frequency is 100 Hz and the sample rate is 48 kHz.](../../.gitbook/assets/butterworth-lowpass.png)

![Butterworth high filter frequency response as a function of filter order. The filter order goes from 1st order \(least steep line\) to 10th order \(steepest line\). The cutoff frequency is 100 Hz and the sample rate is 48 kHz.](../../.gitbook/assets/butterworth-high-filter-pass.png)

**SecondOrderFilterSmoothed**

This module is the most frequently used filter among all of the Audio Weaver modules. It implements a 2nd order Biquad filter and includes design equations for 20 different filter types. The filter type and high-level design parameters \(frequency, gain, and Q\) can be changed at run-time using the inspector:

![](../../.gitbook/assets/85%20%281%29.png)

Depending on the filter type, some parameters are not used. See the table below for the filter types available and which control parameters are applicable.

* **Pass Through**
  * filterType = 0
  * Applicable parameters: none.
  * Biquad coefficients are set to b0=1, b1=0, b2=0, a1=0, and a2=0. The filter runs and consumes processing but the output equals the input.

![](../../.gitbook/assets/pass-through.png)

**Gain**

* filterType = 1
* Applicable parameters: gain
* A simple gain with coefficients set to b0=undb20\(gain\), b1=0, b2=0, a1=0, and a2=0

![](../../.gitbook/assets/image%20%2851%29.png)

\*\*\*\*

**1st order Butterworth lowpass filter**

* filterType = 2
* Applicable parameters: freq

![](../../.gitbook/assets/image%20%28104%29.png)

**1st order Butterworth highpass** 

```text

```

* filterType = 4
* Applicable parameters: freq 

![](../../.gitbook/assets/image%20%2888%29.png)

**2nd order Butterworth highpass** 

* filterType = 5
* Applicable parameters: freq

![](../../.gitbook/assets/image%20%2890%29.png)

**1st order allpass** 

* filterType = 6
* Applicable parameters: freq 

![](../../.gitbook/assets/image%20%2838%29.png)

**2nd order allpass** 

* filterType = 7
* Applicable parameters: freq and Q

![](../../.gitbook/assets/image%20%2863%29.png)

**2nd order low shelf** 

* filterType = 8
* Applicable parameters: freq and gain

{% hint style="info" %}
Use as a low frequency tone control
{% endhint %}

![](../../.gitbook/assets/image%20%2874%29.png)

**2nd order low shelf with Q**

* filterType = 9
* Applicable parameters: freq, gain, and Q

![](../../.gitbook/assets/image%20%2880%29.png)



* **2nd order high shelf** 
  * filterType = 10
  * Applicable parameters: freq and gain

{% hint style="info" %}
Use as a high frequency tone control
{% endhint %}

![](../../.gitbook/assets/image%20%2884%29.png)

* **2nd order high shelf with Q** 
  * filterType = 11
  * Applicable parameters: freq, gain, and Q 

![](../../.gitbook/assets/image.png)

* **2nd order peaking / parametric**
  * filterType = 12
  * Applicable parameters: freq, gain, and Q

{% hint style="success" %}
Commonly used for generic equalization since it has controllable frequency, gain, and Q settings.
{% endhint %}

![](../../.gitbook/assets/image%20%2898%29.png)

* **2nd order notch** 
  * filterType = 13
  * Applicable parameters: freq and Q 

![](../../.gitbook/assets/image%20%285%29.png)

* **2nd order bandpass filter** 
  * filterType = 14
  * Applicable parameters: freq and Q 

![](../../.gitbook/assets/image%20%2847%29.png)

* **1st order Bessel lowpass filter** 
  * filterType = 15
  * Applicable parameters: freq 

![](../../.gitbook/assets/image%20%2823%29.png)

* **1st order Bessel highpass filter**
  * filterType = 16
  * Applicable parameters: freq 

![](../../.gitbook/assets/image%20%2853%29.png)

* **1st order asymmetrical low shelf** 
  * filterType = 17
  * Applicable parameters: freq and gain

![](../../.gitbook/assets/image%20%2833%29.png)

* **1st order asymmetrical high shelf** 
  * filterType = 18
  * Applicable parameters: freq and gain 

![](../../.gitbook/assets/image%20%2826%29.png)

* **1st order symmetrical low shelf**
  * filterType = 19
  * Applicable parameters: freq and gain 

![](../../.gitbook/assets/image%20%2862%29.png)

* **1st order symmetrical high shelf** 
  * filterType = 2
  * Applicable parameters: freq and gain

![](../../.gitbook/assets/image%20%2835%29.png)



{% hint style="info" %}
The Butterworth filter from SecondOrderFilterSmoothed is the same as the ButterworthFilter module of equal filter order. However, SecondOrderFilterSmoothed only implements 1st and 2nd order Butterworth filters. **Higher order Butterworth filters can only be implemented by the ButterworthFilter module.**

The SecondOrderFilterSmoothed implementation of the first order Butterworth filter is more computationally efficient than the ButterworthFilter module.

Low/high shelf filter and low/high shelf filter Q are identical if Q is set to 0.707 \( \).
{% endhint %}



**Second Order Filter Smoothed Cascade**

This module contains several SecondOrderFilterSmoothed modules in series. This can be used to implement a more complicated EQ with only a single module. Under module properties, specify the number of stages of filtering. If the number of stages is set to 1, then this module is equivalent to the SecondOrderFilterSmoothed module. 

![When there are multiple stages, the inspector expands.](../../.gitbook/assets/112.png)

#### Table of Filter Modules

{% embed url="https://docs.google.com/spreadsheets/d/1XsqvrlftnGJugtKfPGS6ii17InfwarwD0Nv-qS5q0b0/edit?usp=sharing" %}



### Frequency Domain

#### Frequency Domain Modules

Modules for processing signals in the frequency domain are found in the Frequency Domain folder. Frequency domain processing yields novels solutions to audio processing problems and may also lead to more efficient implementations. This section describes the main concepts behind frequency domain processing, then Filterbank Processing describes more sophisticated processing using weighted-overlap short-term Fourier transform filterbanks.

**Complex Data Support**

Audio Weaver natively supports complex data within wire buffers. The data is stored in an interleaved fashion:

```text
real[0], imag[0], real[1], imag[1], real[2], etc
```

For multichannel data the interleaving of real and complex data happens at the lowest level. For example, interleaved stereo data is stored as:

```text
L_real[0], L_imag[0], R_real[0], R_imag[0], L_real[1], L_imag[1], R_real[1], R_imag[1], etc.
```

Two modules are provided to convert between real and complex data

|  | Block Name | Description |
| :--- | :--- | :--- |
| ![](../../.gitbook/assets/114.png) | RealImagToComplex | Converts two real signals into complex data using one as the real part and the other as the imaginary part |
| ![](../../.gitbook/assets/115.png) | ComplexToRealImag | Converts a complex signal into separate real and imaginary components |

The system below essentially does nothing except convert two mono signals into complex and then back again. If view wire info is enabled, \(“ViewData type”\) it will mark complex wires with a “C”.

![](../../.gitbook/assets/116.png)

**Transform Modules**

Audio Weaver provides 3 different transform modules for converting between the time and frequency domains.

|  | Icon | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/117.png) | Cfft | Complex FFT. Supports both forward and inverse transforms |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/118.jpeg) | ![](../../.gitbook/assets/119.png) | Fft | Forward FFT of real data |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/120.jpeg) | ![](../../.gitbook/assets/121.png) | Ifft | Inverse FFT yielding real data |

The complex FFT takes a complex N-point input and generates a complex N-point output. The module is configured on the module properties as either a forward or inverse transform.

![](../../.gitbook/assets/122.png)

The Fft and Ifft modules are designed to operate on real signals. The Fft modules takes an N-point real input and generates an N/2+1 point complex output. The output signal contains frequency samples from DC \(\) all the way up to and including the Nyquist frequency \(\). A property of the real FFT is that the samples at DC and Nyquist contain real data only and the imaginary components are guaranteed to be zero. _These samples are still stored as complex values but the imaginary component is zero._ The output of the real FFT will therefore consist of the samples:

```text
X[1]         complex
X[2]         complex
…
X[N/2-1]     complex
X[N/2]       real
```

The Ifft takes N/2+1 complex samples and returns a real N-point sequence. The Ifft ignores the imaginary component of the DC and Nyquist samples.

**Windowing**

Before an FFT is computed the signal is typically windowed to prevent edge effects from influencing the results. There are 3 modules which perform windowing.

| Icon | Name | Description |
| :--- | :--- | :--- |
| ![](../../.gitbook/assets/123.png) | Window | Simple Window |
| ![](../../.gitbook/assets/124.png) | WindowOverlap | Window with overlapping |
| ![](../../.gitbook/assets/125.png) | WindowAlias | Windowing followed by time aliasing |

The windowing modules are for advanced users who use Matlab to compute window coefficients.

The Window module can compute a large number of different window functions. Under module properties, specify the length of the window to apply. Then on the inspector, specify the starting and ending indexes of the window as well as the window type and an optional amplitude.

![](../../.gitbook/assets/126.png)

Allowing the ability to change the starting and ending indexes of the window is more flexibility than is usually needed.

The WindowOverlap module has an internal FIFO that buffers up data into overlapping blocks. For example, a 64-sample input block size with a 50% overlap turns into 128 sample blocks, to be windowed. Essentially, the WindowOverlap module contains a Rebuffer module combined with a Window module. The module has an internal array of window coefficients. This array is initialized to a Hamming window \(raised cosine\) at instantiation time. To change the window coefficients use the Matlab scripts.

The WindowAlias module applies a window followed by time aliasing the sequence to a shorter length. This module is used in the analysis back of short-term Fourier transform based filterbanks.

| Icon | Name | Description |
| :--- | :--- | :--- |
| ![](../../.gitbook/assets/127.png) | OverlapAdd | Reduces block size by overlapping blocks |

The OverlapAdd module performs the opposite of the Rebuffer module. The module has a large input block size and a smaller output block size. The module contains an internal buffer equal to the input block size. The module takes the input data, adds it to the internal buffer, and then shifts out one block of output data. The data in the internal buffer is also left shifted and the leading samples are filled with zeros. The OverlapAdd module finds use in fast convolution algorithms.

| ![](../../.gitbook/assets/128.png) | RepWinOverlap | Replicates data, applies a window, and then performs overlap add |
| :--- | :--- | :--- |


The RepWinOverlap module is for advanced users building synthesis filterbanks. The module replicates a signal N times, applies a window, and then performs overlap add.

| ![](../../.gitbook/assets/129.png) | ZeroPad | Adds zeros at the end of a buffer |
| :--- | :--- | :--- |


The ZeroPad module inserts zeros at the end of a signal. Specify the length of the output buffer under module properties. If the output is longer than the input then the signal is zero padded. If the output is shorter than the input then the signal is truncated.

**Complex Math**

The frequency domain modules have a large number of modules which operate on complex data. The modules here are listed without detailed explanations because the underlying functions are basic and easily understood.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Icon</th>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/130.png" alt/>
      </td>
      <td style="text-align:left">ComplexAngle</td>
      <td style="text-align:left">Computes atan2 of complex data</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/131.png" alt/>
      </td>
      <td style="text-align:left">ComplexConjugate</td>
      <td style="text-align:left">Conjugates data by negating the imaginary component</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/132.png" alt/>
      </td>
      <td style="text-align:left">ComplexMagnitude</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/134.png" alt/>
      </td>
      <td style="text-align:left">ComplexModulate</td>
      <td style="text-align:left">Multiplies by &#x1D452;&#x1D457;&#x1D714;&#x1D458;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/135.png" alt/>
      </td>
      <td style="text-align:left">ComplexMultiplier</td>
      <td style="text-align:left">
        <p>Complex x Complex, or</p>
        <p>Real x Complex</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/136.png" alt/>
      </td>
      <td style="text-align:left">ComplexToPolar</td>
      <td style="text-align:left">Converts to Polar (angle and magnitude)</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/137.png" alt/>
      </td>
      <td style="text-align:left">PolarToComplex</td>
      <td style="text-align:left">Converts from Polar to Real/Imag</td>
    </tr>
  </tbody>
</table>The modules listed above operate on complex data only. A few of the other Audio Weaver modules found outside the Frequency Domain folder are also able to operate on complex data type:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BlockConcatenate</td>
      <td style="text-align:left">Combines blocks of complex data</td>
    </tr>
    <tr>
      <td style="text-align:left">BlockDelay</td>
      <td style="text-align:left">Delays by multiples of the block size</td>
    </tr>
    <tr>
      <td style="text-align:left">BlockExtract</td>
      <td style="text-align:left">Extracts a portion of the complex data</td>
    </tr>
    <tr>
      <td style="text-align:left">BlockFlip</td>
      <td style="text-align:left">Frequency flips data</td>
    </tr>
    <tr>
      <td style="text-align:left">Deinterleave</td>
      <td style="text-align:left">Pulls apart multichannel complex signals into individual mono complex
        signals</td>
    </tr>
    <tr>
      <td style="text-align:left">Demultiplexor</td>
      <td style="text-align:left">Outputs complex data to one output pin; zeros the rest</td>
    </tr>
    <tr>
      <td style="text-align:left">Interleave</td>
      <td style="text-align:left">Combines multiple mono complex signals into a single multichannel complex
        signal</td>
    </tr>
    <tr>
      <td style="text-align:left">Multiplexor</td>
      <td style="text-align:left">Selects one of N complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">ShiftSamples</td>
      <td style="text-align:left">Left or right shifts complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">Adder</td>
      <td style="text-align:left">Adds two complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">ClipAsym</td>
      <td style="text-align:left">Clips the real and imaginary components</td>
    </tr>
    <tr>
      <td style="text-align:left">Invert</td>
      <td style="text-align:left">Multiplies by + or -1. Set <code>smoothingTime = 0</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">Mixer</td>
      <td style="text-align:left">Mixers together complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">MixerDense</td>
      <td style="text-align:left">Mixers together complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">MuteSmoothed</td>
      <td style="text-align:left">Multiplies by +1 or 0. Set <code>smoothingTime = 0</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScaleOffset</td>
      <td style="text-align:left">Scale both the real and imaginary components and adds an offset</td>
    </tr>
    <tr>
      <td style="text-align:left">ScalerDB</td>
      <td style="text-align:left">dB gain without smoothing</td>
    </tr>
    <tr>
      <td style="text-align:left">Scaler</td>
      <td style="text-align:left">Linear gain without smoothing</td>
    </tr>
    <tr>
      <td style="text-align:left">Subtract</td>
      <td style="text-align:left">
        <p>Subtracts two complex signals</p>
        <p></p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SumDiff</td>
      <td style="text-align:left">Adds and subtracts complex signals</td>
    </tr>
    <tr>
      <td style="text-align:left">WhiteNoise</td>
      <td style="text-align:left">Generates uncorrelated noise in both real and imaginary components</td>
    </tr>
    <tr>
      <td style="text-align:left">ScalerDBControl</td>
      <td style="text-align:left">dB gain with gain value taken from a control pin. Set <code>smoothingTime = 0</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScalerControl</td>
      <td style="text-align:left">Linear gain with the gain value taken from a control pin. Set <code>smoothingTime = 0</code>.</td>
    </tr>
  </tbody>
</table>

####  FilterBank Processing

**Introduction**

This Section describes the filterbank blocks. The blocks are based on a weighted overlap-add \(WOLA\) design and are applicable to a wide range of audio processing tasks. The document first describes how the blocks work from an end user’s point of view. It then describes the theory behind the filterbanks and how they lead to efficiency during runtime.

**Using WOLA and sub-band Blocks**

The WOLA filterbank blocks are part of the DSPC Concepts IP Folder. The Frequency Domain contains the key set of Audio Weaver modules which are used for performing frequency domain computations. There are blocks for FFTs, windowing, complex operations, etc. Frequency domain operations often involve filterbanks, and Audio Weaver also includes modules for implementing entire weighted overlap-add filterbanks. There are separate modules for the forward filterbank \(the analysis bank\) and the inverse filterbank \(the synthesis bank\).

The blocks are called “WOLA Analysis” and “WOLA Synthesis”. When dragged out, they will appear as follows in the layout:

![](../../.gitbook/assets/138.png)

The input to the WOLA Analysis bank is real time domain data and the output is complex frequency domain data. Similarly, the input to the WOLA Synthesis bank is complex frequency domain data and the output is real time domain data. When configuring the filterbanks using Module Name and Arguments, the FFT size \(K\) and the stopband attenuation between subbands is specified. This holds for both the analysis and the synthesis banks. Under module name and arguments, this would show:

![](../../.gitbook/assets/139.png)

The FFT specifies the number of frequency domain “bins” and the input \(and output\) block size is always ½ of the FFT size. For example, if using a 32 sample block size will only work with an FFT size K = 64. Manually set this on both the analysis and the synthesis filterbanks. This will error out if improperly specified:

![](../../.gitbook/assets/140.png)

The attenuation relates to the separation between outputs of the filterbank, in dB, and will be described in more detail later in the guide. A “safe” value to use is somewhere in the range from 40 to 80 dB. When combining analysis and synthesis filterbanks, ensure that the same value of attenuation is used throughout.

Assuming a block size of 32, set the FFT size K = 64. Making connections between blocks and then showing wire sizes:

![](../../.gitbook/assets/141.png)

Note that the output of the filterbank contains 33 complex samples rather than 64. This is because the filterbank modules use _real_ FFTs and as a result discard the redundant conjugate symmetric data. Only K/2+1 bins are kept, which in this case equals 33. The bins have the following properties:

```text
Bin k=0.     Real data.
Bin k=1.     Complex data.
Bin k=2.     Complex data.
…
Bin k=31.    Complex data
Bin k=32.    Real data
```

The first and last bins have real data; this is a property of the FFT and results from the fact that the input data is real. Audio Weaver stores the output of the FFT as 33 complex values with the imaginary parts of bins k=0 and k=32 set to zero.

The filterbanks accept any number of channels of input data, but it is not a typical scenario in Audio Weaver.

{% hint style="info" %}
Although the analysis and synthesis filterbanks accept any number of channels, most modules in the Frequency Domain folder only operate on mono signals. It is recommended to design systems with mono frequency domain data for greatest flexibility.
{% endhint %}

![](../../.gitbook/assets/142.png)

The text below the filterbank modules also shows the latency through the filterbanks, in samples. The latency is the combined latency through the analysis and synthesis filterbanks given the current values of K and attenuation. Increasing K or increasing the attenuation increases the latency through the filterbanks. use the displayed latency to time align other signals in the system. For example, to check the reconstruction properties of the filterbanks, compensate using a sample delay module:

![The output of the WOLA synthesis bank equals the input of the analysis bank but delayed by 352 samples. In the example above, this latency is compensated with a delay, so the output of the subtract module is essentially zero.](../../.gitbook/assets/143.png)

The output of the WOLA synthesis bank equals the input of the analysis bank but delayed by 352 samples. In the example above, this latency is compensated with a delay, so the output of the subtract module is essentially zero.

This example shows the meter module with a residual difference at around -80 dB. The filterbanks are not perfect reconstruction but introduce a small amount of aliasing noise. The level of aliasing noise is directly related to the attenuation setting of the filterbanks.

The frequency domain outputs of the analysis filterbank represent the outputs of a series of bandpass filters. There are K filters and the spacing between bins is $$2π/K$$ radians, or if the sample rate of the system is SR, then the spacing between bins is $$SR/K$$ Hz. For example, if the sample rate of the system is 48 kHz and K=64, then the spacing between bins is 750 Hz. The first bin \(with real data\) is centered at 0 Hz. The next bin is centered at 750 Hz, and so on. The last bin \(with real data\) is centered at 24 kHz.

The filterbanks also contain built in decimation. The outputs of the analysis bank represent the _decimated_ outputs of bandpass filters. The decimation factor equals the block size, that is, K/2. Continuing the example from above, the sample rate of the system is 48 kHz and the block size is 32 samples. Thus, the sample rate of the frequency domain subbands is 1500 Hz. see this by showing the sample rate on the wires.

**Theory**

This section describes more of the mathematical theory behind the filterbanks. The design of the filterbanks was based primarily on chapter 7 of the book _Multirate Digital Signal Processing_ by Crochiere and Rabiner. This is an excellent and very readable introduction to the subject of filterbanks. Our description follows the derivation found in this book.A classical filterbank uses a time domain window function followed by an FFT as shown below: ![](../../.gitbook/assets/144.png)

![](../../.gitbook/assets/screen-shot-2020-03-11-at-11.59.26-pm.png)

The length of the FFT equals the length of the window function. In many cases, the window function is a raised cosine, or Hanning window:

![](../../.gitbook/assets/145.png)

The input blocks of the filterbank are overlapped in time. There are many ways to describe the amount of overlapping. The terminology “50% overlap” indicates that from FFT to FFT, K/2 new input samples are made. If there is “75% overlap” then there are K/4 new samples for each FFT computed. In this discussion, the phrase “block size” is used to describe how many new samples arrive each time. This approach is also referred to as a short-term Fourier transform \(STFT\).

There are two different ways of looking at the output of the STFT analysis bank. On is to segment the input signal into blocks which are windowed and then FFT’ed. The output of the analysis bank thus corresponds to frequency spectra. On the other hand, a careful study of the analysis bank shows that it is in effect implementing a set of parallel bandpass filters as shown below.

![Analysis filterbank implementation as a parallel set of bandpass filters and decimators.](../../.gitbook/assets/screen-shot-2020-03-12-at-12.00.29-am.png)

The input signal is filtered and then decimated by the block size M. The filters are all related by the mathematical expression

ℎ𝑘𝑛=ℎ0𝑛𝑒𝑗2𝜋𝑘𝑛/𝐾

where is the _prototype lowpass filter_ and all other filters are related to the prototype filter by complex modulation. In the frequency domain, the filters are shifted versions of the prototype filter

𝐻𝑘𝜔=𝐻0𝜔−2𝜋𝑘/𝐾

For example, if a Hanning window is used as the prototype filter,

ℎ𝑛=121−cos2𝜋𝑛𝐾−1

then the frequency response for K = 32 is

![Frequency response of a 32-point Hanning window. The graph shows normalized frequencies in the range 0 to 1.0 which corresponds to 0 to &#x3C0; radians/sample.](../../.gitbook/assets/147.png)

Subsequent bins are spaced by \(or when viewed as normalized frequencies\) and the first 4 bins are shown below:

![](../../.gitbook/assets/148.png)

Note that the prototype filter is quite wide in the frequency domain and there is significant overlap between neighboring bins. Not only does bin k overlap with bin k+1, but also with k+2 and k+3. If a decimation factor of 16 is picked, then aliasing will start at normalized frequency of 1/16 as shown below. The prototype filter has only attenuated the signal by 0.5 and severe aliasing will occur.

![Frequency response of a 32-point Hanning window overlayed with a rectangle indicating where aliasing would occur if the filter output is decimated by a factor of 16](../../.gitbook/assets/149.png)

If the decimation factor is changed to 8, then aliasing begins at a normalized frequency of 1/8 SR and the filter has attenuated the signal. However, with a decimation factor of 8 the 32 sample Hanning window only advances 8 samples each time and this corresponds to an overlap factor of 75%.

![This time the rectangle indicates where aliasing occurs for a decimation factor of 4.](../../.gitbook/assets/150.png)

Is there a way to achieve high decimation while at the same time avoiding aliasing? This brings up the weighted overlap-add filterbank \(WOLA\). The block based derivation from Crochiere and Rabiner avoids aliasing while supplying high decimation. 



![Overview of analysis filterbank implementation](../../.gitbook/assets/screen-shot-2020-03-12-at-12.05.00-am.png)

The main difference is that the prototype filter is N times longer and that after multiplying the input signal, the output is time aliased to the FFT length. Time aliasing is a standard property of the FFT. Suppose an input signal is given: $$r[n]$$of length $$KN$$ . Time alias this to a shorter signal $$x[n]$$ of length $$K$$ 

$$
𝑥𝑛=𝑝=0𝑁−1𝑟[𝑛+𝑝𝐾]
$$

 The FFT$$X[k]$$of $$x[n]$$ is related to the FFT$$R[k]$$of$$r[n]$$by subsampling  

$$
X[k]=R[kN]
$$

That is, $$X[k]$$contains samples of $$R[k]$$ spaced by $$N$$ bins.

The advantage of using a longer prototype filter is that it allows us to get better frequency separation between bands. Consider the designs shown below with N=1, N=2, and N=4. The filters get progressively sharper in frequency and for N=4, the passband of the filter falls within the rectangle indicating the aliasing region for a decimation factor of 16. Thus a high decimation factor is achieved while avoiding high amounts of aliasing.

![](../../.gitbook/assets/152.png)

Now let’s plot the frequency response of the first 4 filters in the filterbank assuming an FFT size of 32 samples, a window length of 128 samples, and a decimation factor of 16.

![ First 3 subband filters for the WOLA filterbank with K=32, N=4, and a decimation factor of 16.](../../.gitbook/assets/153.png)

When N is increased to a very high number to achieve a decimation factor of 32, the result is a critically sampled filterbank with no net increase in data. This limit can be approaced, but never achieved in practice. With realizable filters, a filter will always overlap its immediate neighbors. In Audio Weaver, a decimation factor of K/2 is used and the filterbanks are _oversampled_ by a factor of 2. There is a net doubling of the data rate, but this is important because it decouples the subbands and allows them to be modified without introducing further aliasing distortion.

Recent theory of filterbanks has been focused on critically sampled filterbanks. These filterbanks find use in audio compression and since the goal in compression is to reduce the overall data rate, it is important not to oversample and introduce more data in the subband representation. However, the operations performed on subbands in audio codecs are very gentle compared to what is possible with our WOLA filterbanks. In audio compression, the goal is for the output to equal the input. In Audio Weaver processing systems, the focus is to be able to make gross changes to the subbands without introducing objectionable aliasing artifacts. This requires a fundamentally different approach. Furthermore, if the algo calls for a frame overlap add and overlap save convolution in a filterbank framework, oversampling is needed. _In general, in order to perform subband modifications of audio signals without introducing objectionable aliasing distortion, some amount of oversampling is required._

**Aliasing Performance of the WOLA Filterbanks**

As noted above, the filters in the filterbanks are not ideal and introduce some amount of aliasing. The amount of aliasing depends upon the stopband attenuation used in the design of the filters. This section provides details on the amplitude of this aliasing noise. To test this, use the system shown below:

![](../../.gitbook/assets/154.png)

Analysis and synthesis filterbanks are placed back-to-back. The input is white noise, the output is subtraction of the inputs while compensating for the delay through the filterbanks. Comparing the energy at the input to the energy of the residual noise provides an indication of the level of the aliasing components. The following table shows the aliasing level and latency as a factor of the stopband attenuation of the prototype low pass filter. In the test, an FFT size of 256 samples was used with a resulting blockSize of 128 samples.

| Stopband Attenuation \(dB\) | Measured Aliasing Noise \(dB\) | Latency \(samples\) | Latency \(blocks\) |
| :--- | :--- | :--- | :--- |
| 30 | -28 | 384 | 3 |
| 40 | -39 | 640 | 5 |
| 50 | -50 | 896 | 7 |
| 60 | -61 | 1152 | 9 |
| 70 | -61 | 1152 | 9 |
| 80 | -78 | 1408 | 11 |
| 90 | -87 | 1664 | 13 |

Keep in mind that the aliasing components are linearly related to the input signals. That is, reducing the level of the input signal by 20 dB results in the level of the aliasing components dropping by 20 dB. Thus, the aliasing level is more similar to a signal to noise ratio \(SNR\) rather than total harmonic distortion.

**Subband Signal Manipulation**

Part of the beauty of these filterbanks is that it is possible to manipulate the signals in the subband domain. For example, if scaling the subband signals as shown below, the result will be an equalizer with linearly spaced frequency bins.

![](../../.gitbook/assets/156.png)

Another nice property of the WOLA filterbanks is that they have built in smoothing. That is, making an instantaneous gain change to one of the subband signals then the net effect at the output will be smooth. This is because the synthesis bank has built in low pass filters in each subband and these smooth out discontinuities.

The FIR filter example can be taken further. The example above had only a single gain within each subband. What if the goal is to have more frequency resolution? Place FIR filters into each subband. A longer FIR filter would provide more resolution within that particular frequency band. Consider the following example. A filterbank has an FFT size of 64 samples and is operating with a decimation factor of 32. If the input is 48 kHz then each subband has a sample rate of 1.5 kHz. If an FIR filter of length 500 samples is placed in the DC subband \(bin k=0\), then this yields an effective frequency resolution of 3 Hz within this band. The amount of computation needed to implement this filter is approximately 1500 x 500 = 0.75 MIPs. High resolution is needed in audio applications at low frequencies. For higher frequencies, reduce the lengths of the FIR filters and achieve something close to “log frequency resolution”. By proper design of the subband filters, designing phase response becomes simple.

Any of the Frequency Domain modules which operate on complex data operate in the subband domain. Audio Weaver also provides a special set of “Subband Processing” modules that start with the “Sb” prefix. These modules replicate some of the standard time domain modules but the operations occur separately in each subband.

|  |  |  |  |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/157.png) | SbAttackRelease | Attack and release envelope follower \(real data only\) |
|  | ![](../../.gitbook/assets/158.png) | SbDerivative | Derivative \(real data only\) |
|  | ![](../../.gitbook/assets/159.png) | SbComplexFIR | Complex FIR filter |
|  | ![](../../.gitbook/assets/160.png) | SbNLMS | Normalized LMS adaptive filter |
|  | ![](../../.gitbook/assets/161.png) | SbSmooth | Performs smoothing across subbands \(real data only\) |
|  | ![](../../.gitbook/assets/162.png) | SbRMS | RMS with settable time constant \(real data only\) |
|  | ![](../../.gitbook/assets/163.png) | SbSOF | Second order filter \(real data only\) |
|  | ![](../../.gitbook/assets/164.png) | SbSplitter | Subdivides the spectrum into overlapping regions. Similar to a crossover |

**Synthesis Filterbank**

The synthesis filterbank takes the subband signals and reconstructs a time domain output. **Error! Reference source not found.**Remember that the analysis filterbank can be considered to be a parallel set of bandpass filters and decimators. The synthesis filterbank uses a the inverse of this with upsamplers, filters, and adders. The upsamplers take the decimated subband signals and return them to the original sampling rate by inserting M-1 zeros between each sample value. In the frequency domain, upsampling creates copies of the input spectrum at multiples of and the filters remove the high frequency copies.

![Synthesis filterbank implementation using upsamplers and bandpass filters.](../../.gitbook/assets/screen-shot-2020-03-12-at-1.02.52-am.png)

For efficiency, the synthesis filterbank is implemented using an inverse FFT and periodic replication. As in the analysis filterbank, the window function corresponds to the impulse response of the prototype lowpass filter used in subband k=0.

![Synthesis bank implemented using an inverse FFT followed by windowing and overlapping.](../../.gitbook/assets/screen-shot-2020-03-12-at-1.01.56-am.png)

### Gains

Scaler modules multiply the input signal by a constant value. think of scalers as gain controls or faders. There are many different types of scalers and they have the following characteristics:

1. Smoothed or unsmoothed
2. Linear or dB gains
3. Single gain applied to all channels or individual gains per channel

All scalers support multichannel data and arbitrary block sizes. The scaler modules with a single gain support multiple input and output pins and the number of pins is specified as a constructor argument.

If the scaler has individual gains per channel, then the initial number of channels can be specified as a constructor argument. This makes it easy to enter initial gains before completing the entire block diagram. Thereafter the number of gains is updated by pin propagation. For these modules, the gain values are specified using an array. **The module to use for multiple gain channels is the General Purpose Vector Scaler. It supports smoothing/non, linear/dB, and multichannel gain control.**

In unsmoothed scalers the gain change takes place immediately and may result in an audible “pop” due to the discontinuity. To avoid this, use a scaler with built-in smoothing instead. The following list explains the differences between the deprecated scaler modules, which each had its own function.

{% hint style="warning" %}
#### **THESE MODULES ARE OUT OF DATE, USE GENERAL PURPOSE SCALER**. 

This list is here to keep documentation for the deprecated modules, whose documentation is similar to the fract32 scaler modules.
{% endhint %}

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left"></th>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/167.png" alt/>
      </td>
      <td style="text-align:left">Scaler</td>
      <td style="text-align:left">
        <p>Single linear gain</p>
        <p>Unsmoothed</p>
        <p>Multiple I/O pins</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/168.png" alt/>
      </td>
      <td style="text-align:left">ScalerSmoothed</td>
      <td style="text-align:left">
        <p>Single linear gain</p>
        <p>Smoothed</p>
        <p>Multiple I/O pins</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/169.png" alt/>
      </td>
      <td style="text-align:left">ScalerDB</td>
      <td style="text-align:left">
        <p>Single dB gain</p>
        <p>Unsmoothed
          <br />Multiple I/O pins</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/170.png" alt/>
      </td>
      <td style="text-align:left">ScalerDBSmoothed</td>
      <td style="text-align:left">
        <p>Single dB gain</p>
        <p>Smoothed</p>
        <p>Multiple I/O pins</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/171.png" alt/>
      </td>
      <td style="text-align:left">ScalerN</td>
      <td style="text-align:left">
        <p>Per channel linear gains</p>
        <p>Unsmoothed</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/172.png" alt/>
      </td>
      <td style="text-align:left">ScalerNSmoothed</td>
      <td style="text-align:left">
        <p>Per channel linear gains</p>
        <p>Smoothed</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/173.png" alt/>
      </td>
      <td style="text-align:left">ScalerNDBSmoothed</td>
      <td style="text-align:left">
        <p>Per channel dB gains</p>
        <p>Smoothed</p>
      </td>
    </tr>
  </tbody>
</table>For the above modules, the gain to apply is specified by an inspector variable. The module library also has two controllable scalers. For these modules, the gain to apply is taken from the first input pin. There are linear and dB variants and both have built-in smoothing.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/174.png) | ScalerControl | Controllable scaler with linear gain |
|  | ![](../../.gitbook/assets/175.png) | ScalerDBControl | Controllable scaler with dB gain |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/176.jpeg) | ![](../../.gitbook/assets/177.png) | Invert | Smoothing scales between +1 and -1 and provides a phase inversion. |
|  | ![](../../.gitbook/assets/178.png) | ScaleOffset | Multiplies the signal by a fixed scale factor and adds an offset. |

These modules are frequently used to invert or add an offset to a signal by setting the scale factor equal to 1.0. This is easier than using separate DCSource and Adder modules.

#### Mutes

Several other modules exist which do smoothing scaling of signals between fixed values. The MuteSmoothed module scales between 0 and 1 and the Invert module scales between +1 and -1. There is also a MuteNSmoothed module which is designed for multichannel signals and allows the individual mute control for each channel.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/179.jpeg) | ![](../../.gitbook/assets/180.png) | MuteSmoothed | Smoothly mutes and unmutes a signal. |
|  | ![](../../.gitbook/assets/181.png) | MuteNSmoothed | Multichannel mute with separate controls for each channel. Similar to ScalerNSmoothed. |

The SoloMute module allows muting of all channels except one \(like soloing on a soundboard\). The SoloMute module can have multiple input pin, where each input can have multiple channels. Or, if it has a single input pin, the soloing functionality is applied to individual channels.

|  | ![](../../.gitbook/assets/182.png) | SoloMute | Similar to MuteN except that it includes the ability to solo or listen to only 1 channel. |
| :--- | :--- | :--- | :--- |


#### Crossfader

The Crossfader module takes in two sources and smoothly varies between the two input sources. It also has energy levelling during the crossfade based on either a Linear scale \(amplitude\) or the energy of the signal \(rms\). This module has a control pin that varies between 0 and 1, correlating to the next two input pins. It is limited to two channels.

#### General Purpose Scalers

As mentioned earlier, the general purpose scaler supports smoothing, dB and linear gain values, optional input pin, and even pin counts. If a multichannel-multivalued gain is needed, the General Purpose Vector Scaler supports up to 256 simultaneous channels.

#### Table of Mute Modules

| Module Name | Input type supported | Smoothing | Per Channel Control |
| :--- | :--- | :--- | :--- |
| Mute N Smoothed | Floating Point | Yes | Yes |
| Mute Smoothed | Floating Point | Yes | No |
| Mute Unmute | Floating Point | Yes | No |
| Solo Mute | Floating Point | Yes | Yes |
| Synchronous Mute | Floating Point | Yes | No |

#### Table of Gain Modules

{% embed url="https://docs.google.com/spreadsheets/d/1GZXR1xEdeZC8O-lMmGQth33oevIGKxO3F8Pnnm98z0g/edit?usp=sharing" %}

### Logic

#### Control Signals and Boolean Logic

Control signals are defined as having a block size of one. This useful for reducing the computational load when only a single value is needed. Wires holding control signals are shown as dashed lines \(![](../../.gitbook/assets/185.png)\).

To illustrate the use of control signals we’ll build up a Fletcher-Munson equal-loudness contour. This guide is not intended to go into depth on topics of audio processing, but in simplest terms, this system adapts frequency as the level of audio drops. The human ear perceives both the low- and high-range frequencies dropping off more quickly than the mid-range frequencies. A common method for dealing with this in audio processing is to boost the bass and treble frequencies as the volume drops to create what is called an “equal-loudness contour,” or a “Fletcher-Munson curve.” This can be achieved by a filter which varies based on the desired volume level. This example will demonstrate the process, including the use of several control signals.

The complete system \(shown below\) has a few key points. As the volume is reduced at the DCSource, the levels of the bass and treble frequencies will be boosted so that they sound as though they are dropping off at the same rate as the mid-range frequencies. Note the dotted lines representing all the control signals. This implementation is also computationally efficient. The control signals have a block size of 1 and do not require much computation. The main processing is performed by the Scaler and two SOFControl modules.

![](../../.gitbook/assets/186.png)

The VolumeSetting module is a DCSource which outputs the gain setting, in dB; this is a control signal and is drawn as a dotted line. When configuring the VolumeSetting using module properties set its blockSize and number of channels to 1.

The first step is to reduce the signal level by VolumeSetting. This is accomplished using the ScalerDBControl module. This module takes its gain setting from an input pin rather than from an inspector and allows for a control signal-dependent gain.

|  | ![](../../.gitbook/assets/187.png) | ScalarDBControl | Simple dB scalar; gain controlled by control signal input |
| :--- | :--- | :--- | :--- |


In this example it will be used to allow a DC source to adjust the overall volume. Thus begins the system as shown below:

![](../../.gitbook/assets/188.png)

Another example of a control signal module is the SOFControl module.

|  | ![](../../.gitbook/assets/189.png) | SOF Control | Second-order filter with gain determined by control signal input |
| :--- | :--- | :--- | :--- |


The SOFControl module allows for a control signal to adjust parameters \(frequency, gain, or Q\) of a second order filter. The control parameter\(s\) are selected in the module’s module properties. In this case, only the gain is controlled as shown below:

![](../../.gitbook/assets/190.png)

In this example, this will allow for a volume-dependent bass \(and treble\) boost. The SOFControl module will take in the audio from the Scalar module as one of its inputs \(the lower input pin\):

![](../../.gitbook/assets/191.png)

![](../../.gitbook/assets/192.png)

Since the purpose of this filter is to boost bass frequencies, a filter type of “Peak EQ” and a frequency of 30 Hz are selected in the module’s inspector. The upper input pin of the SOFControl module requires a control signal input. This control input will originate from the DCSource controlling the volume. To achieve the equal-loudness contour, the bass must be boosted in relation to the volume. The mapping between the volume and the bass boost is accomplished with the TableInterp module:

|  | ![](../../.gitbook/assets/193.png) | TableInterp | Second-order filter with gain determined by control signal input |
| :--- | :--- | :--- | :--- |


The TableInterp module allows the user to map out the intended input-output relation visually and interpolates between given points to produce a continuous function. In this case, the relation shown below will be sufficient:

![](../../.gitbook/assets/194.png)

At full volume \(0 dB\) there is no bass boost. As the signal level drops more boost is progressively applied. At -80 dB 28 dB of boost is applied. Also note that the table interpolation both takes in and outputs a control signal. Only a single value is translated through the table’s function at a time.

A sink module can be appended to the output of the table interpolation to show in real-time the computed gain. Note that when the sink module is given a control signal as input, it will only display the single value, rather than the graph it normally displays.

![](../../.gitbook/assets/195.png)

This has accomplished the boosting of the bass frequencies, and the same strategy can be employed to boost the treble frequencies. This SOFControl will be set to alter the treble frequencies, with a filter type of “Shelf High” and a frequency of 6000 Hz:.

This TableInterp will use a slightly different relation but will achieve fundamentally the same function – boosting the level at lower volumes:

![](../../.gitbook/assets/196.png)

**Boolean Signals**

A Boolean signal has only two possible values, 0 and 1. Boolean data is useful for controlling systems. Boolean signals are represented using 32-bit integer values. Audio Weaver includes several modules that perform logical operations on Boolean signals.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/197.png) | BooleanInvert | Inverts a Boolean signal \(logical NOT gate\) |
|  | ![](../../.gitbook/assets/198.png) | LogicAll | Resolves to 1 if and only if all its inputs are non-zero \(logical AND gate\) |
|  | ![](../../.gitbook/assets/199.png) | LogicAny | Resolves to 1 if any of its inputs are non-zero \(logical OR gate\) |

By default the LogicAll and LogicAny modules do not have any output wires; they store the output in an internal variable \(“.result”\). By checking the box next to “outputValue” in the module properties, an output pin can be created.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/200.png) | LogicBinaryOp | Performs binary operations on Boolean data |

The LogicBinaryOp module allows the user to select a logical operation \(logical AND, OR, and XOR\). It takes two control signals as input and outputs one wire with the computed Boolean value.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/201.png) | BooleanSource | Source module with Boolean data output |

The BooleanSource module is a source module that supplies a buffer of Boolean data. As with any source module, the number of channels, block size, and sample rate are user-specified in the module properties.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/202.png) | LogicCompare | Compares input values |
|  | ![](../../.gitbook/assets/203.png) | LogicConstCompare | Compares input to constant |

The LogicCompare module performs one of many possible comparisons on two input values. In its Inspector is a drop-down menu of the possible comparisons: EQUAL, NOTEQUAL, LESSTHAN, LESSOREQUAL, GREATERTHAN, and GREATEROREQUAL.

![](../../.gitbook/assets/204.png)

The module performs the comparison and outputs 1 if the comparison resolves to true and 0 if it resolves to false.

The LogicConstCompare module functions the same way, except it compares a single input to a constant, user-specified value.

A simple example of the use of the Boolean signals is shown below. This system selects the louder of the two input channels and outputs it on both channels. To accomplish this, the RMS module measures the levels of the two input signals and outputs control signals. These control signals are then fed into a LogicCompare control, set to LESSTHAN comparison type. Its output controls the index of the MultiplexorFadeControl module which selects the louder of the two input channels.

![](../../.gitbook/assets/205.png)

If RMS1 &lt; RMS2 then the LogicCompare module outputs 1 and the louder right channel is selcted; if RMS1 &gt;= RMS2 then the module outputs 0 and the louder left channel is selected.

**ParamSet and ParamGet**

Control signals can be used to adjust parameters of modules. This can also be accomplished with the Param Get and Param Set modules. These are found in the Misc. folder. The example from the previous section can be made even cleaner by using ParamSet.

![](../../.gitbook/assets/206.png)

The output of the LogicCompare module is fed straight into the ParamSet module, which in turn sets the index of the MultiplexorFade module. Note that in this version, the multiplexor does not have to be a control module.

Just like ParamGet, the target parameter is specified in the module properties, with “name of the module”.“parameter to be set”. Also specify the data type of the input wire. In its Inspector, the ParamSet module allows the user to change the when the parameter update occurs using the setBehavior drop-down menu:

![](../../.gitbook/assets/207.png)

The choices available are:

`AlwaysNoSet` – Always update the variable in the instance structure. Do not call the module’s set function.

`AlwaysDeferredSet` – Always update the variable in the instance structure. Call the module’s set function in the deferred processing thread.

`OnChangeNoSet` – Only update the variable when it has changed. Do not call the module’s set function.

`OnChangeDeferredSet` – Only update the variable when it was changed. Call the module’s set function in the deferred processing thread.

`OnChangeInstantSet` – Only update the variable when it has changed. Then call the module’s set function from the real-time thread. Change occurs immediately.

The many options allow for optimization of computational efficiency by performing the change at a time when it is most appropriate for the system. In most cases, the `OnChangeDeferredSet` is the most appropriate.

### Math

#### Advanced Math

The advanced math modules include the functions seen below:

![](../../.gitbook/assets/208.png)

The Convolve module is similar to the FIR module, with the property that it can ignore signal tails. It is normally not used for filtering. Its shape parameter acts as a truncation control. When the user selects shape 0, no truncating occurs, outputting normal convolution. If the user selects shape 1, the module will ignore the first N/2 samples of the output, then display the next N samples, followed by ignoring the last N/2 samples. This is useful for statistics between two data sets. This is essentially partial correlation.

The Correlate module is similar to the convolve module, except it differs in the order of the output.

Derivative and Integral modules compute discrete derivative and integral equations respectively:

$$
y[n] = 1/dt * (x[n] - x[n-1])
$$

 where $$dt$$ is the time step, $$dt = 1/SR$$.

$$
y[n] = dt/K * sum(x[0] .. x[n])
$$

 where $$dt$$ is the time step, $$dt = 1/SR$$ and $$K$$ is a gain.

#### Basic Math

These modules perform basic math operations.

|  |  |  |  |
| :--- | :--- | :--- | :--- |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/209.jpeg) | ![](../../.gitbook/assets/210.png) | Adder | Adds signals |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/209.jpeg) | ![](../../.gitbook/assets/212.png) | Subtract | Subtracts signals |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/209.jpeg) | ![](../../.gitbook/assets/214.png) | Multiplier | Multiplies signals |
|  | ![](../../.gitbook/assets/215.png) | Divide | Divides signals |
|  | ![](../../.gitbook/assets/reciprocal.png)  | Reciprocal | Computes 1/x |

The Adder and Subtract module by default have two input pins. Additional pins can be specified on the module properties. For the Adder, all inputs are summed together. For the Subtract module, the last input pin is subtracted from the others. The Adder and Subtract modules both handle signals with multiple input channels.

The Adder module has an additional property that can be selected on the module properties:

![](../../.gitbook/assets/216.png)

When oneChannelOutput is checked then the output will be mono and all input channels will be summed together to form the output. In this mode, the input pins can have _different numbers of channels._ By default, oneChannelOutput is unchecked and all input pins have to have the same number of channels and the output will be multichannel as well. A useful way of using the Adder is to sum together the left and right components of a stereo signal. The traditional way is to use a Deinterleave module

![](../../.gitbook/assets/217.png)

Or, use an Adder module with the number of input pins set to 1 and oneChannelOutput checked:

![](../../.gitbook/assets/218.png)



In addition to these basic math functions, the Math folder has modules corresponding to the functions in the standard C math.h library. These modules will be skipped, but here is a preview:

![](../../.gitbook/assets/220.png)

#### DB Conversion

To convert to and from dB10 and dB20, this folder hosts Approx and exact modules. The Approx modules are less cpu intensive, and less accurate. To be clear, the UndB modules convert from dB to linear scaling.

![](../../.gitbook/assets/221.png)

#### Lookup Tables

![](../../.gitbook/assets/222.png)





The Table Interp module uses a table with clickable/movable points to discern how the input values scale into the output pin. For data in between the points, the scaling is interpolated either linearly or cubicly

![](../../.gitbook/assets/2%20%283%29.png)

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>The Table Interp module uses a table with clickable/movable points to
          discern how the input values scale into the output pin. For data in between
          the points, the scaling is interpolated either linearly or cubicly</p>
        <p>
          <img src="../../.gitbook/assets/2 (3).png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/224.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>
          <img src="../../.gitbook/assets/226.png" alt/>
        </p>
        <p>Table Interp2d takes in a vector input (x and y pins) and scales them
          according to the nPoints length matrix. It uses bilinear interpolation
          of the table values at the four neighboring points (above, below, left,
          right).</p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/225.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>
          <img src="../../.gitbook/assets/229.png" alt/>
        </p>
        <p>TableLookup uses either Linear or nearest interpolation. The user has
          to specify the upper and lower bounds of this table. Set the size of the
          table with the &#x2018;L&#x2019; variable found in the arguments:</p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/227.png" alt/>
        <img src="../../.gitbook/assets/228.png" alt/>
      </td>
    </tr>
  </tbody>
</table>

For non-interpolated lookup tables, an integer index is listed, and output is based on that index’s data value. That data can be float or int type data.

![TableLookupIntFloat](../../.gitbook/assets/230.png)

#### 

![TableLookupIntInt](../../.gitbook/assets/231.png)

#### Nonlinearities

Modules which implement point nonlinearities and are stateless and easy to understand. They are listed below and used in the examples later on.

|  |  | Name | Description |
| :--- | :--- | :--- | :--- |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/232.jpeg) | ![](../../.gitbook/assets/233.png) | ClipAsym | Hard clipper |
|  | ![](../../.gitbook/assets/234.png) | Polynomial | Polynomial f\(x\) |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/235.jpeg) | ![](../../.gitbook/assets/236.png) | Square | Squares a signal x\*x |

Other nonlinearities which are documented elsewhere are the Abs, TableInterp, SoftClip, and TwoPieceInterp modules.

#### Trig

Trig functions are simple enough to not need much detail. They operate in radians, not degrees. The following trig functions are provided:

#### ![](../../.gitbook/assets/237.png)

### Misc

Most of these modules are used for debugging, and verification during the product development phase. These modules are great for testing the hardware, stimulating conditions, profiling, and adding custom interfacing with the Board Support Package such as GPIO and other \(non-audio\) input signals. There are also modules in here that help get around the strict data flow of AudioWeaver by manipulating how the data is represented \(i.e. samples turn into channels, etc\).

#### Biquad Loading

This module builds virtual biquads in memory with a number of max stages, and a variable that “turns on” the filter function. As data passes through, the mips and memory will update in the server window. This is useful for evaluating what kind of processing works and fits on a target. 

![](../../.gitbook/assets/239.png)

Set maxStages in the Arguments tab.

![](../../.gitbook/assets/243.png)

#### Block Counter

This module outputs the number of blocks processed based on the system BlockSize. This source increments for each block called at the hardware defined samplerate. It then outputs this value. This is useful for determining how many blocks get processed in the system, typically for debugging.

![](../../.gitbook/assets/240.png)

#### Coeff Table

![](../../.gitbook/assets/241.png)

This module is similar to paramSet, except it sends a vectored variable rather than a singular numeric variable. Set the coeffs to be sent to the variable by typing into the properties sheet, then clicking apply\(see right\). Upon clicking apply, the coefficient data is sent to whatever module lies within the modVar argument. \(see below\)

![](../../.gitbook/assets/242.png)

#### Counter

This counter module counts at a given interval, with a minimum interval of one msec. This is good for debugging, as it gives a time constant to determine when\(in ms\) an issue arises.

#### Cycle Burner

This module is similar to the Biquad loading, except it works in computation cycles per block, and has no inputs or outputs. This is convenient for testing out multiple block sizes and cpu load for the target.

#### FIRLoading

This module is similar to the Biquad loading, except it is more memory intensive. Use this to determine the amount of FIR modules that can be loaded onto a target.

#### GPIO

The GPIO module is binary only. It can both accept GPIO signals and send them to the target, based on parameters in the module properties. The sample rate can also be specified. This should match the setup in the BSP. Most target chips will have dedicated GPIO lanes, and will specify how to set it up.

#### Math Exception

This module replaces NAN and INF with the value specified by the user. This is useful for keeping stability in a system which may require numeric data.

#### Measurement

This module is used to measure a room based on a periodic output of the input signal. This is used to average the recorded signal into the response buffer. The trigger pin starts the measurement upon changing from 0 to 1. Once the measurement is complete \(based on numReps and the length of the signal, L\), the trigger is set back to 0.

#### Param Get

The ParamGet module reads the value of a parameter from an existing module and outputs it on a wire for use elsewhere in the system. Which parameter to output is specified in the module properties, with “name of the module”.“parameter to be output”. An example of this module can be found in ParamSet and ParamGet.

|  | ![](../../.gitbook/assets/244.png) | ParamGet | Outputs a parameter from a specified module |
| :--- | :--- | :--- | :--- |


#### 

![ParamGet Inspector](../../.gitbook/assets/243.png)

#### Param Set

|  | ![](../../.gitbook/assets/245.png) | ParamSet |  | Sets a parameter of a specified module |
| :--- | :--- | :--- | :--- | :--- |


The ParamSet module does the opposite: it takes an input from the system and sets its value to a parameter of an existing module. The key is that this module can set parameters of any type of module, which eliminates the need to create separate controllable versions of each module. An example of this module can be found in ParamSet and ParamGet.

![](../../.gitbook/assets/246.png)

#### Safety Clip

This module clips an incoming signal based off of an external trigger. The clip amount can be specified in the properties. This is used in systems to prevent damage to equipment by digitally clipping loud sources. This usually results in less gain given.

#### Sample Rate

This module counts the sample and outputs the estimated sample rate of a system. This is useful for debugging samplerate related issues.

#### Set Wire Properties

This module is used to change wire properties for a more direct handling of audio weaver data values. It works by treating the samples as different block sizes, channels, data type, sample rate, and complexity. The only rule is that the samples in must equal the samples out. For example, at 48k and a block size of 32 stereo signal, that means there are 32 samples every block, or 64 samples per block of stereo information. These 64 samples can also be represented as blocksize=64, one channel. Notice the wire info and SetWireProperties modules in the following design:

![](../../.gitbook/assets/247.png)

While this doesn’t give much advantage in the time domain with standard audio data, this module is especially strong in the frequency domain to handle signal flow.

#### Status Set

This module can change the “status” of other modules in the design. This is useful for limiting compute cycles during runtime. To bypass a module during runtime, the status set module is given a variable name of where to point, and four controls for the status states: Active, Muted, Bypass, and Inactive. The input takes in an integer signal 1-4, with any other number gets treated as 1. These numbers represent the four states. If this control signal gets bypassed, then the status set module cannot turn itself back on. The diagram below is an example of a very low power “processing” system, which takes advantage of the status set module.

![](../../.gitbook/assets/248.png)

#### Update Sample Rate

This module updates the sample rate information for wires and modules. It does not change the sample rate. For that, see the FIR Interpolator and Decimator in the “Multirate” folder.

### Mixers

Mixers combine separate audio sources into fewer channels, or into each other.

#### MixerV3

A Mixer is an M x N array of gains that transform M input channels into N output channels. This is useful for combining signals together, and consolidating many gain values into a single module. Our Mixer module is smoothed, and optimized based on the amount of non-zero coefficients specified by the user. It supports input pin controls or single pin, multi-channel controls. A mixer is good for handling many channels at once, so it excels in the algorithm for downmixing from surround sound 5.1 to stereo. This means transforming 5 input channels to 2 output channels:

![](../../.gitbook/assets/250.png)



The Mixer module holds a matrix representing the downmix equations, which are shown in its inspector \(in linear units\):

![](../../.gitbook/assets/249.png)

```text
Lt = L + -3dB*C + -3dB*(-Ls -Rs)

Rt = R + -3dB*C + -3dB*(Ls + Rs),
```

The Mixer module stores a matrix of all the gain coefficients and performs matrix multiplication to apply the gains to the appropriate channels. When the matrix becomes large, this computation becomes very expensive. It may be beneficial to deinterleave out channels if they don’t require the mixing stage.

#### SMixer2x1

This mixer takes in two pins of arbitrary channel count and outputs the sum of the two with a scale factor specified in the module’s variables.

![SMixer2x1](../../.gitbook/assets/251.png)

#### Wet Dry

This module is similar to the 2x1 mixer, except the two pins are exclusively balanced from one another to achieve either same energy or same amplitude. This setting can be configured in module variables. The linear mixing value represents the ratio from 0-1 of the mixing of the signals. A value of .5 means equal mix from both signals, and is similar to the Smixer2x1 module. Values under .5 mean that the signal is less “wet” so more of the “dry” pin comes through. Values over .5 drive more of the “wet” pin.

### Multirate

#### Multirate Processing

Audio Weaver is able to process signals at different samples rates all within the same layout. This was seen with control signals but the feature is much more powerful. Audio Weaver is able to handle multirate processing in two different ways:

1. Single block time processing
2. Multiple block time processing

In single block time processing all of the audio modules execute within a single thread at the same rate. In multiple block time processing there are multiple threads on the target processors and different block times execute within separate interrupt levels. Each approach is described in turn.

**Single block time processing**

A module has an associated block size and sampling rate. We’ve been treating these wire properties as separate information but when combined they yield the _block time_ of a module. For example, consider the system shown below.

![](../../.gitbook/assets/252.png)

The sampling rate is 48 kHz and the block size is 32 samples. Each block of audio thus represents 32 / 48000 = 2/3 msec of audio. 32 sample audio buffers arrive every 2/3 msec and each block executes in turn.

There are 4 modules which can be used to change the sampling rate and still maintain the same block time.

|  |  |  |  |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/253.png) | Upsampler | Inserts zeros between samples. No filtering |
|  | ![](../../.gitbook/assets/254.png) | Downsampler | Discards samples. No filtering. |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/255.jpeg) | ![](../../.gitbook/assets/256.png) | FIRInterpolator | Upsampler followed by an FIR interpolating filter |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/257.jpeg) | ![](../../.gitbook/assets/258.png) | FIRDecimator | FIR filter followed by a Downsampler |

The Upsampler and Downsampler modules insert zeros and discard samples, respectively. Specify the up and downsampling factors on the module properties. The downsampling factor must be chosen so that it divides the input block size and yields an integer number of output samples. Consider the system shown below:

![](../../.gitbook/assets/259.png)

As before, the input block time is 2/3 of a millisecond. The Upsampler module is configured for an upsampling factor of L=2. The output sampling rate is 96 kHz and the block size is now 64 samples. Note that 64/96000 = 2/3 millisecond and the block time is preserved. As before, all modules execute every 2/3 millisecond.

The Upsampler and Downsampler modules correspond to standard up and downsamplers found in DSP text books. Since they lack any filtering, they are rarely used.

The FIRInterpolator and FIRDownsampler modules, on the other hand, contain FIR filters. The FIRInterpolator inserts zeros and then filters the resulting signal with a lowpass filter; the FIRDecimator first applies an FIR lowpass filter and then decimates. Both modules use an efficient polyphase implementation to reduce the processing load. These modules also preserve the underlying block time.

On the module properties for the FIRInterpolator / FIRDecimator specify the up / downsampling factor as well as the length of the FIR filter. The length of the FIR filter must be an integer multiple of the up / downsampling factors. When the modules are instantiated the FIR filter coefficients are computed using a Hamming window. Advanced users can change the filter coefficients by using Matlab scripts.

![](../../.gitbook/assets/260.png)

The Rebuffer module stores and overlaps buffer data into larger block sizes, allowing for more data to be displayed. It does not change the fundamental block size for the system.

|  | ![](../../.gitbook/assets/261.png) | Rebuffer | Overlaps data into larger block sizes, allowing for longer time displays. Does not change fundamental block size |
| :--- | :--- | :--- | :--- |


The Rebuffer can accept data of any type. In its module properties is a variable called “outBlockSize,” which allows the user to set the output block size for the module. If a positive value is entered, that value is used as the output block size. If a negative value is entered, the value is used as a multiplier to the input block size. For example, an outBlockSize of 32 will yield an output of block size 32, and an outBlockSize of -8 yields an output with 8 times the block size of the input.

For example, the following block diagram shows a SinGen source wired directly to a Sink module to view in the scope display:

![](../../.gitbook/assets/262.png)

However, the scope display shows only a small amount of data \(in this case, 0.67 msec, based on the example above with a block size of 32 and sample rate of 48000 Hz\):

![](../../.gitbook/assets/263.png)

To extend the amount of displayed data, a Rebuffer module with outBlockSize -8 is added between the source and the sink:

![](../../.gitbook/assets/264.png)

This allows the Sink module to display 8 times the amount of data, easily spanning the 4-msec window:

![](../../.gitbook/assets/265.png)

The Rebuffer module is also useful for multirate processing. The Rebuffer module increases the output block size while keeping the sampling rate and block time constant. It achieves this by outputting blocks which _overlap in time._ Consider the system shown below. The input block size is 32 samples and the Rebuffer module is configured to output 128 sample blocks.

![](../../.gitbook/assets/266.png)

Each block that is output overlaps the previous one by 96 samples as shown below.

![](../../.gitbook/assets/screen-shot-2020-03-12-at-12.43.49-am.png)

The Rebuffer module is useful for frequency domain processing when it is necessary to have a certain amount of overlap between blocks. The inverse of the Rebuffer module is the BlockExtract module. This module extracts a subset of samples and reduces the block size.

**Multiple block time processing**

In some applications processing needs to be performed at multiple block times. Consider a system that has low latency processing with a block size of 32 samples combined with frequency domain processing at a block size of 256 samples. At a 48 kHz sampling rate, the 32 sample processing would occur every 2/3 millisecond while the 256 sample processing would occur every 5 1/3 millisecond. This type of processing is achieved using the BufferUp and BufferDown modules.

|  | Name | Description |
| :--- | :--- | :--- |
| ![](../../.gitbook/assets/268.png) | BufferUp | Buffers up to larger blocks without overlapping. |
| ![](../../.gitbook/assets/269.png) | BufferDown | Buffers down to smaller blocks without dropping samples. |

The BufferUp module generates larger non-overlapping blocks. On the module properties dialog specify the output block size either as an integer number of samples or as a multiple of the input block size. In the example above, to go from 32 to 256 samples, specify a 256 sample block size \(or a multiplier of 8\). To return to a 32 sample block size, use the BufferDown module. Again, explicitly specify the output block size either as an integer number of samples or as a divider.

The system shown below combines 32 and 256 sample block sizes.

![](../../.gitbook/assets/270.png)

In the figure, the name of the filter module is shown as “SOF1 \[1/8\]”. The annotation \[1/8\] indicates that this module executes in a separate thread at a rate of 1/8th compared to the others. The BufferUp module indicates \[1/1 1/8\] which means that the part of the module operates at the full rate \(1/1\) and part of the module executes at the 1/8 rate. The output of the BufferUp module contains 256 samples which equals 8 32-sample blocks. The output is _non-overlapping_ as shown here.

![](../../.gitbook/assets/screen-shot-2020-03-12-at-12.39.50-am.png)

The BufferUp and BufferDown modules contain internal double buffering to connect the two processing rates. The double buffering introduces a latency equal to twice the larger block size. In the example above, the latency through the BufferUp, SOF, and BufferDown modules equals 512 samples.

On the target processor, the 32 and 256 sample processing occurs in different threads \(or interrupt levels\). The 32 sample processing occurs in a higher interrupt level and actually interrupts the 256 sample processing. The pattern of processing would be as shown below. The 32 sample block processing occurs at a uniform rate. When the 32 sample processing is not active then the 256 sample processing has a chance to execute. The 256 sample processing must complete before the next 8 blocks of 32 samples arrive.

![](../../.gitbook/assets/screen-shot-2020-03-12-at-12.39.22-am.png)

Running at multiple block times can lead to erroneous module profiling results for the larger block time. The small block time is correct but the large block time incorrectly includes the time needed to execute the smaller blocks. Be aware of this when viewing profiling results.

Another limitation of Audio Weaver is that the smallest possible block time in the system corresponds to the fundamental block size of the target system. This means that the smallest block time occurs at the _input pin_ of the system. Audio Weaver buffers can only BufferUp to larger block times. When using a platform with a fundamental block size of 32 samples, it is not possible to BufferDown to a 1 sample block size for stream processing.

One final note, not all targets support multiple block times. Refer to the user guide of the specific hardware target to see if this feature is supported.

### Signal Management

#### Signal Routing

Signal routing modules are modules that manipulate the flow of data. This is done on a few different levels. The marker module is a simple fix to _wiring_ components overlapping, and has no effect on the runtime algorithm. This module can also be used as test points to view signal response. There are also modules that control _channel flow_, allowing interleave/deinterleave, or even the routing of one channel into another, and finally the multiplexing to choose between various signals. _Data type conversion_ occurs when the samples are all converted into another numerical format. The more complicated modules in this folder control _block flow_ and _sample flow_.

**Multiplexors**

Multiplexors \(or Muxes\) are logic elements that allow different signals to be selected based on an index \(or control\) value. Muxes are useful for setting up A/B comparisons to allow accurate comparison of two different signals. Multiplexors support multichannel signals and the number of input pins \(number of signals to select from\) is specified as a constructor argument.

There are 3 different types of muxes:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/273.png" alt/>
        <img src="../../.gitbook/assets/274.png" alt/>
      </td>
      <td style="text-align:left">Multiplexor (MultiplexorV2 &amp; MultiplexorV2Fract32)</td>
      <td style="text-align:left">Supports smoothing for switches and crossfading into/from silence. Index
        taken from inspector or from an optional control pin. Float and Fract32
        versions.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/275.png" alt/>
      </td>
      <td style="text-align:left">Multiplexor Unsmoothed (Multiplexor)</td>
      <td style="text-align:left">
        <p>Just copies data; no smoothing</p>
        <p>Supports any 32-bit data type</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/276.png" alt/>
      </td>
      <td style="text-align:left">Sample Multiplexor Control</td>
      <td style="text-align:left">Sample based mux, no smoothing. Only has control pin</td>
    </tr>
  </tbody>
</table>{% hint style="success" %}
If the signals that are switched between is a control signal or a signal where discontinuity is not important, then choosing `0 smoothing` or `0 fade time`has computational advantages.
{% endhint %}



![](../../.gitbook/assets/278.png)

The index variable is zero based and determines which signal is selected. If there are two signals to select from, then the inspector is drawn as a checkbox. Unchecked \(zero\) selects the first signal and checked \(one\) selects the second signal.

![](../../.gitbook/assets/279.png)

If there are more than two signals to select from the inspector has a drop list 

![](../../.gitbook/assets/280.png)

Here are a few examples of how to use a multiplexor module in practice. MultiplexorFade performs seamless crossfading between signals. In this first example, the multiplexor is being used to enable an optional equalizer.



![](../../.gitbook/assets/281.png)





In the next example, 2 different versions of an algorithm are compared to see which one sounds better. This is a classical “A/B” comparison in audio.

![](../../.gitbook/assets/282.png)

**Marker**

The Marker module is used label wires in the design and also to make wire routing prettier. The Marker module is a virtual mode and is removed from the system when it is built. The Marker module is very frequently used. Add a Marker module to the layout by right-clicking on an empty portion of the canvas and selecting Add Marker.

![](../../.gitbook/assets/283.png)

Below is an example of how to use the marker module to clean up a design. Initially the wire goes through the text label of a module making it difficult to read.

![](../../.gitbook/assets/284.png)

Now reroute the wires with as few or as many markers as needed.

![](../../.gitbook/assets/285.png)

![](../../.gitbook/assets/286.png)

The second use of the Marker module is in computing frequency responses. **get the frequency response of an individual filter by right-clicking on it and selecting “Plot Frequency Response”**. With a chain of filters, insert Marker modules at the input and output wires.

![](../../.gitbook/assets/287.png)

Then using the Measurement menu plot the frequency response between Markers M1 and M2. This is given in Tools-&gt;Measurements.

![](../../.gitbook/assets/288.png)

The measurements made will be saved here, allowing for easy updating of plots.

![](../../.gitbook/assets/289.png)

Finally, the plot will pop up in its own window with the standard matlab plot capabilities, as well as an “update” button to redraw the frequency or phase response of the system. Use this to see how filters are responding with each other. This is based on the filter coefficients, so this feature doesn’t work on every module. For reading the output of modules, use the sink modules.

![](../../.gitbook/assets/290.png)

**Interleave/Deinterleave**

Two of the most basic modules are the Interleave and Deinterleave modules.

| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/291.jpeg) | ![](../../.gitbook/assets/292.png) | Deinterleave | Turns multichannel signals into separate mono signals. |
| :--- | :--- | :--- | :--- |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/293.jpeg) | ![](../../.gitbook/assets/294.png) | Interleave | Combines multiple mono channels into a single interleaved signal. |

These modules are part of the default system which is created when starting a new design. For the Deinterleave module specify the number of output channels on module properties and it must match the number of input channels. If there is a mismatch an error will pop up when the system is built.

The Interleave module takes N input signals and combines them into a single multi-channel output. The number of input pins is specified by module properties. One special property of the Interleave module is that not all input pins have to be mono. In fact, any number of channels per input pin converts to the output simply “stacking” all of the input channels.

In the first example, a stereo input’s left channel is delayed by 10 msec.

![](../../.gitbook/assets/295.png)

In the next example two stereo signals \(L/R and Ls/Rs\) are combined together with two mono channels \(C and LFE\) to form a 6 channel output. The channels in the interleaved output pin will be ordered: L / R / Ls / Rs / C / LFE.

![](../../.gitbook/assets/296.png)

**Router**

The Router module simply copies input channels to output channels. The module solves many common signal management issues like selecting or recombining channels and in most cases is more efficient than using Interleave and Deinterleave modules.

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <img src="../../.gitbook/assets/297.jpeg" alt="http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg"
        />
      </th>
      <th style="text-align:left">
        <img src="../../.gitbook/assets/298.png" alt/>
      </th>
      <th style="text-align:left">Router</th>
      <th style="text-align:left">
        <p>Copies and combines audio channels.</p>
        <p>No smoothing.</p>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/299.png" alt/>
      </td>
      <td style="text-align:left">RouterSmoothed</td>
      <td style="text-align:left">
        <p>Copies and combines audio channels.</p>
        <p>With smoothing.</p>
      </td>
    </tr>
  </tbody>
</table>

In its most general form, the Router module has M input pins and N output channels. The numbers M and N are specified on the module properties:

![](../../.gitbook/assets/301.png)

For simplicity, we’ll start with a single input pin and then build up to the more complicated case of multiple input pins.

![](../../.gitbook/assets/300.png)

In order to use the Router module effectively, how the channel routing map is specified must be understood. The module has a .channelIndex array of length N, where N is the number of output channels. The input channel to copy to the nth output channel is specified by channelIndex\[n\] and the channels are ordered starting with 0. Let’s start with a simple example. Suppose that a stereo signal needs the right channel extracted. This can be accomplished with two modules as shown below. The NullSink module is used to tie off the unused left channel output.

![](../../.gitbook/assets/303.png)

Alternatively, the same function can be accomplished with a single router module. Configure it for a single input pin \(M=1\) and a single output channel \(N=1\). Then set channelIndex\[0\] = 1. This copies the right channel \(the second channel is chosen due to the index set to 1\) to the first output channel.

![](../../.gitbook/assets/302.png)

![](../../.gitbook/assets/304.png)

In the second example, a sound card with 12 input channels is used. The 2 target audio channels are contained in the last two input channels. In this case, configure the router module for 1 input pin and 2 output channels. 



![Set the channelIndex array as shown](../../.gitbook/assets/305.png)

For the final example, crossfeed between the left and right channels. That is…

𝐿𝑜𝑢𝑡=𝐿𝑖𝑛+0.25𝑅𝑖𝑛

𝑅𝑜𝑢𝑡=𝑅𝑖𝑛+0.25𝐿𝑖𝑛

The router module can be used to swap the order of the left and right channels making it easy to implement the crossfeed utilizing stereo processing.

![](../../.gitbook/assets/306.png)

Consider the case of a router module with multiple input pins. Each element of the channelIndex\[\] array is treated as a 32-bit unsigned integer and packs in a pin\# and channel\# as follows:

```text
channelIndex[n] = (pinNum << 16) + channelNum
```

That is, the pin number is in the high 16-bits and the channel number is in the low 16 bits. Thus far, all examples have had a single input pin starting at pinNum = 0 setting the focus on channelNum, some positive integer offset.

Suppose that there are two wires holding 5.1 channel data. The channels are ordered as L0 / R0 / Ls0 / Rs0 / C0 / LFE0 in the first wire and L1 / R1 / Ls1 / Rs1 / C1 / LFE1 in the second wire. The goal is to form a new 5.1 channel signal by combining the signals as L0 / R0 / Ls1 / Rs1 / C0 / LFE0. This can be accomplished with a single router module as follows.

![](../../.gitbook/assets/307.png)

The values in the channelIndex array are computed as[\[5\]]():

```text
Pin 0, Channel 0 -->    (0 << 16) + 0    =    0
Pin 0, Channel 1 -->    (0 << 16) + 1    =    1
Pin 1, Channel 2 -->    (1 << 16) + 2    =    65538
Pin 1, Channel 3 -->    (1 << 16) + 3    =    65539
Pin 0, Channel 4 -->    (0 << 16) + 4    =    4
Pin 0, Channel 5 -->    (0 << 16) + 5    =    5
```



{% hint style="info" %}
The notation \(pin &lt;&lt; 16\) represents a left shift by 16 bits. Alternatively this equals \(pin \* 65536\).
{% endhint %}

The Router module can handle any 32-bit data type. The channelIndex array can be changed at run-time and the change in router occurs instantly without smoothing. For a smoothly varying channel router without pops or clicks, use the RouterSmoothed module.

### Sinks

Sink modules have an input pin but no output. Sinks are used to tie off unused module pins or for plotting of immediate data.

|  |  |  |  |
| :--- | :--- | :--- | :--- |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/308.jpeg) | ![](../../.gitbook/assets/309.png) | NullSink | Ties off unused module outputs. No internal buffer. |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/310.jpeg) | ![](../../.gitbook/assets/311.png) | Sink | Copies Floating point data from the wire to an internal buffer. Can be used as a scope display. |
|  | ![C:\Users\peaston\AppData\Local\Temp\Clipboarder.2016.10.11-002.png](../../.gitbook/assets/312.png) | Boolean Sink | Triggers a light \(via inspector\) on if “true” and off if “false” |
|  | ![](../../.gitbook/assets/313.png) | Triggered Sink | Sink that copies data when input control is nonzero, else it does nothing. |
|  | ![C:\Users\peaston\AppData\Local\Temp\Clipboarder.2016.10.11-003.png](../../.gitbook/assets/314.png) | Sink Int | Copies Int Data from wire to internal buffer. Can be used as a scope display. |

#### Null Sink

One of the examples from above used a NullSink module to tie off an unused output pin:

![](../../.gitbook/assets/315.png)

The left channel output is attached to a NullSink module and completely ignored. The NullSink does no processing and uses very little memory \(only for its instance structure\).

#### Sink

The Sink module, on the other hand, has an internal buffer equal in size to the input wire. At run-time, the Sink module copies the data from the input wire to the internal buffer. This takes some processing and some memory. The interesting feature of the Sink module is that the inspector has a scope display that shows the real-time wave form.

If the Sink module is displaying a signal with a block size of 1 then the inspector just shows the value \(no waveform\). This is useful for displaying the contents of control signals.

![](../../.gitbook/assets/316.png)

The Sink module shown above is in a system that has a 32 sample block size and displays less than a millisecond of data. The Sink module is often used in conjunction with the Rebuffer module to display longer time sequences. For example, Rebuffer[\[6\]]() the data into a 1024 sample block and then attach this to a Sink module.

![](../../.gitbook/assets/317.png)

The waveform shows would be about 23 msec in length as shown below and also shows two channels of data.

![](../../.gitbook/assets/318.png)

#### Meter

If only an instantaneous view of a system value is desired, the Meter module can be used instead.

| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/319.jpeg) | ![](../../.gitbook/assets/320.png) | Meter | Shows instantaneous dB level in inspector; has multichannel input and separately meters each channel |
| :--- | :--- | :--- | :--- |


The Meter module’s instantaneous display is shown in its inspector and the inspector expands to include multiple channels.

 ![](../../.gitbook/assets/321.png) ![](../../.gitbook/assets/322.png)

The ClipIndicator module also provides instantaneous feedback in real-time, but it only displays whether or not audio is being clipped \(i.e. the signal exceeds 0 dB\).

|  | ![](../../.gitbook/assets/323.png) | ClipIndicator | Shows instances of clipping in real-time |
| :--- | :--- | :--- | :--- |


A black box in the inspector indicates no clipping is occurring:

![](../../.gitbook/assets/324.png)

A red box alerts the user to clipping:

![](../../.gitbook/assets/325.png)

One of the examples from above used a NullSink module to tie off an unused output pin:

![](../../.gitbook/assets/326.png)

The left channel output is attached to a NullSink module and completely ignored. The NullSink does no processing and uses very little memory \(only for its instance structure\).

### Spatial

This module folder hosts the balance module, used for arranging a stereo space using L and R controls.

### Statistics

The RunningMin/Max module can be used to find minimum and maximum of values being processed in real-time. The module keeps track of the min and max values seen over all time and the process can be restarted using a checkbox on the inspector.

|  | ![](../../.gitbook/assets/327.png) | RunningMin/Max | Stores the minimum and maximum values seen thus far. Can output the value, but this is off by default |
| :--- | :--- | :--- | :--- |


This module can output the minimum and maximum values as well as storing and displaying them in the inspector, but this is off by default. It can be turned on by selecting the checkbox next to “outputValue – output type” in the module’s module properties:

![](../../.gitbook/assets/328.png)

The RunningMinMax module is particularly useful for tuning dynamics processors. To ensure that a signal does not exceed a set level, the RunningMinMax module can be configured to display the peak level seen over time.

### Subsystem

This folder holds the subsystem module, as well as input and output pins for that module. These SysIn and SysOut modules should be placed within the subsystem. Their names can be changed, and will be displayed on the subsystem as this name. Subsystems are used to keep a system abstracted and clean. They can be copied and pasted, retaining all variables and module arguments. Finally, they can be bypassed in order to bypass all modules within.

### Sources

Sources have no input but output a signal to be used elsewhere in the system. The output block size, number of channels, and sample rate are constructor arguments. They can be entered manually in the module properties, but will default to those of the system input pin if left empty. If a source module is used within a subsystem then the properties of the source module will be based on the first input pin of the subsystem. The parameters of the source can be changed at runtime via its inspector.

{% hint style="danger" %}
Sources are often very loud \(0 dB\) and must be followed by a scalar to reduce the level.
{% endhint %}

The following source modules are available in the Source folder:

|  |  | Name | Sine wave generator with no smoothing |
| :--- | :--- | :--- | :--- |
|  | ![](../../.gitbook/assets/329.png) | SineGen |  |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/330.jpeg) | ![](../../.gitbook/assets/331.png) | SineSmoothedGen | Sine wave generator with smoothing |
|  | ![](../../.gitbook/assets/332.png) | Source | Continuously copies data from internal buffer to output wire |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/333.jpeg) | ![](../../.gitbook/assets/334.png) | DCSource | Continuously outputs DC value to output wire |
|  | ![](../../.gitbook/assets/335.png) | ImpulseSource | Periodically outputs an impulse; period specified in samples |
|  | ![](../../.gitbook/assets/336.png) | ImpulseMsecSource | Periodically outputs an impulse; period specified in msec |
|  | ![](../../.gitbook/assets/337.png) | PeriodicSource | Continuously outputs a stored array of data. The internal buffer length is specified by module properties. |
|  | ![](../../.gitbook/assets/338.png) | PulseGen | Generates periodic rectangular pulse, with period and on time specified in msec |
|  | ![](../../.gitbook/assets/339.png) | Randi | Generates slowly varying random noise with linear interpolation between sample values |
|  | ![](../../.gitbook/assets/340.png) | Sawtooth | Generates a sawtooth wave \(ramps up linearly from 0 to 1, then repeats\) with specified frequency |
|  | ![](../../.gitbook/assets/341.png) | WhiteNoise | Generates uniformly distributed white noise |
| ![http://images.clipartpanda.com/clipart-star-Star-clip-art-5.jpg](../../.gitbook/assets/342.jpeg) | ![](../../.gitbook/assets/343.png) | PinkNoise | Generates pink noise by applying an IIR filter to internally generated white noise |
|  | ![](../../.gitbook/assets/344.png) | Oscillator | Generates oscillations with specified frequency and start phase |



A table of sample waveform outputs from some of the most basic sources follows:

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Sources</b>
      </th>
      <th style="text-align:left">waveform outputs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p><b>SineGen / SineSmoothedGen</b>
        </p>
        <p>
          <img src="../../.gitbook/assets/345.png" alt/>
          <img src="../../.gitbook/assets/346.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/347.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>Sawtooth</b>
        </p>
        <p>
          <img src="../../.gitbook/assets/348.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/349.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>WhiteNoise</b>
        </p>
        <p>
          <img src="../../.gitbook/assets/350.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/351.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>PinkNoise</b>
        </p>
        <p>
          <img src="../../.gitbook/assets/352.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/353.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>DCSource</b>
        </p>
        <p>(outputs one steady value)</p>
        <p>
          <img src="../../.gitbook/assets/354.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/355.png" alt/>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>Randi</b>
        </p>
        <p>
          <img src="../../.gitbook/assets/356.png" alt/>
        </p>
      </td>
      <td style="text-align:left">
        <img src="../../.gitbook/assets/357.png" alt/>
      </td>
    </tr>
  </tbody>
</table>









