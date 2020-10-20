---
description: General purpose periodic function generator
---

# PeriodicFunctionGen

General purpose function generator, with parameters for controlling frequency, amplitude and start phase. The module can generate Sine waves, Square waves, Triangle waves, Sawtooth waves and Impulses with the variable functionType.

### Arguments

![](../../../.gitbook/assets/0%20%2831%29.png)

* `SR` - Sample rate, default is blank. If blank the sample rate is inherited from the system input.
* `blockSize` - Block size, default is blank. If blank the sample rate is inherited from the system input.

### Variables

![](../../../.gitbook/assets/1%20%2835%29.png)

* `functionType` - Used to select waveform type. There are five choices. Numbers are used to select wave type in Matlab API, text is used to select wave type in awe\_designer:
  * \(0, `SineWave`\) - Generates a sine wave.
  * \(1, `SquareWave`\) - Generates a square wave.
  * \(2, `TriangleWave`\) - Generates a triangle wave.
  * \(3, `SawtoothWave`\) - Generates a sawtooth wave.
  * \(4, `Impulse`\) - Generates an impulse wave.
* `freq` - The frequency of the function generator output wave, in Hz.
* `Amplitude` - The linear amplitude of the output wave.
* `Offset` - The starting phase of the output wave in degrees.

## Examples

### Impulse Example

In the signal flow below the PeriodicFunctionGen module has been set up to produce a periodic impulse at a frequency of 15 Hz. The impulse train is then passed through a low pass butterworth filter with a cutoff of 100 Hz. The resulting impulse response is plotted in the Sink1 inspector. Note that there is not a lot of energy in the impulse below 100 Hz.

![](../../../.gitbook/assets/2%20%2830%29.png)

## Inspector

The inspector for this module is displayed by double clicking on the module in AWE\_designer. The amplitude, freq, and phase offset variables can be changed in real time.

![](../../../.gitbook/assets/3%20%2827%29.png)

## Matlab Usage

`M=periodic_function_gen_module(NAME, SR, BLOCKSIZE)`

Creates a general purpose function generator for use in the Audio Weaver environment. This module has a single channel output pin with controllable frequency. The output of the module changes instantly when the frequency is updated. This module can generate sine wave, square wave, triangular wave, sawtooth wave and impulse with control variables.

###  Arguments

*  `NAME` - name of the module.
*  `SR` - sample rate.
*  `BLOCKSIZE` - number of samples per output channel.

## Type Definition

```cpp
typedef struct _ModulePeriodicFunctionGen
{
   ModuleInstanceDescriptor instance; // Common Audio Weaver module instance
                                      // structure.
   INT32 functionType;                // Type of the wave form to be generated on
                                      // output.
   FLOAT32 freq;                      // Frequency of the function generator output
                                      // wave.
   FLOAT32 amplitude;                 // Amplitude in linear units.
   FLOAT32 offset;                    // Starting phase of the function generator
                                      // output wave.
   FLOAT32 offsetRad;                 // Offset in radians.
   FLOAT32 phase;                     // Instantaneous phase and also starting 
                                      // phase.
   FLOAT32 phaseInc;                  // Instantaneous sample to sample phase
                                      // increment.
   INT32 impPeriod;                   // Period or rate of the impulse generator.
   INT32 impSampleIndex;              // Specifies the index of the next non-zero
                                      // value.
   FLOAT32 triPhase;                  // Instantaneous phase and also starting
                                      // phase.
   FLOAT32 triPhaseInc;               // Instantaneous sample to sample phase
                                      // increment.
   FLOAT32 sawPhase;                  // The stored phase of the sawtooth function.
   FLOAT32 sawPhaseIncrement;         // The amount that the oscillator phase is
                                      // incremented for each output sample.
} ModulePeriodicFunctionGenClass;
```

## See Also

Oscillator, PulseGen, PulseGenFract32, SawtoothFract32, SineGenControlFract32, SineGenFract32, SineSmoothedGen, SineSmoothedGenFract32

