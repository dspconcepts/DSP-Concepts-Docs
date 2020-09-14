# API Quick Reference

## Summary

A processor’s software interacts with the Audio Weaver Runtime Core \(AWE Core\) by exchanging three types of data; Audio, Tuning, and Control.

![](.gitbook/assets/2%20%282%29.png)

THE AWE Core processes _Audio_ data in blocks of 32, 64, or some other power-of-two size. This Audio data is typically exchanged via DMA or shared-memory buffer. It is required to be left-justified, signed, fixed point data \(Q1._n_\).

_Tuning_ data configures the topology of the signal-processing and adjusts tuning parameters such as filter coefficients or channel gain. In the development environment, this data comes from the Audio Weaver Server, which translates human-readable commands \(issued by AWE Designer or 3rd party tuning/design tools\) into the binary protocol expected by the Runtime Core’s Tuning API. In a stand-alone environment, this tuning data comes from a data file \(Audio Weaver Binary, .AWB\) stored in-product, typically in FLASH.

_Control_ is typically either HMI settings, such as volume & mute, or system-state, such as battery-level. Calculated values such as the RMS signal-level, event-detection flags, or a direction of arrival estimates may also be retrieved from the control interface.

This document specifies the API for these three interfaces.

## Audio Interface

### Overview

On each DMA interrupt \(or posted semaphore in the case of audio delivered via shared memory\), blocks of audio samples are copied to the AWE Core’s input-buffers on a channel by channel basis. The number of these channel buffers and the address of each are acquired through API functions listed below. The same holds true for AWE Core’s output buffers, where blocks of audio data are generated on a per-channel basis.

|  |  |
| :--- | :--- |
| void  | **awe\_fwInit** \(AWEInstance \* pAWE\) |
|  | Framework initialization function. |
|  |  |
| INT32  | **awe\_fwGetInputBlockSize** \( AWEInstance \*pAWE, UINT32 pinIdx\) |
|  | Returns the block size of the currently instantiated Layout. |
|  |  |
| INT32  | **awe\_fwGetInputSampleRate** \( AWEInstance \*pAWE, UINT32 pinIdx\) |
|  | Returns the sample rate of the currently instantiated Layout. |
|  |  |
| void  | **awe\_fwGetChannelCount** \( AWEInstance \*pAWE, INT32 \*inCount, INT32 \*outCount\) |
|  | Returns the number of input & output channels in the currently instantiated Layout |
|  |  |
| INT32 \*  | **awe\_fwGetInputChannelPtr** \( AWEInstance \*pAWE, INT32 chan, INT32 \*stride\) |
|  | Returns the address where the next block of input audio samples should be copied. |
|  |  |
| INT32 \*  | **awe\_fwGetOutputChannelPtr**\( AWEInstance \*pAWE,INT32 chan, INT32 \*stride\) |
|  | Returns the address from which processed, output audio samples may be read. |
|  |  |
| INT32  | **awe\_fwPump** \( AWEInstance \*pAWE, UINT32 layout\_no\) |
|  | Executes the currently instantiated Layout, ‘pumping’ audio from input to output. |
|  |  |
| INT32  | **awe\_fwTick** \( AWEInstance \*pAWE\) |
|  | Must be called periodically to allow to operate its internal state machines. |

|  |  |
| :--- | :--- |
| UINT32  | **g\_master\_heap** \[\] |
|  | Master heap - typically L1 RAM. Used for AWE Core state and audio buffers. |
|  |  |
| UINT32  | **g\_slow\_heap** \[\] |
|  | Slow heap - typically L3 RAM. |
|  |  |
| UINT32  | **g\_fastb\_heap** \[\] |
|  | Fast heap - typically L1 RAM. Used primarily for coefficient buffers. |
|  |  |
| UINT32  | **g\_master\_heap\_size** |
|  | Master heap size in DWords. |
|  |  |
| UINT32  | **g\_slow\_heap\_size** |
|  | Slow heap size in DWords. |
|  |  |
| UINT32  | **g\_fastb\_heap\_size** |
|  | Fast heap size in DWords. |
|  |  |
| DWORD  | **g\_target\_control\_flags** = 0 |
|  | Set of target flags for various features. More... |

### Detailed Description

#### Functions

**void awe\_fwInit\( AWEInstance \* pAWE\);**

Initialize the Framework

\[in\] **pAWE** AWE instance pointer \(this\)

**UINT32 awe\_fwGetInputBlockSize \( AWEInstance \*pAWE,**

**UINT32 pinIdx**

**\);**

Returns the number of channels in the Layout's input and output pins.

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **pinIdx** which input pin

**UINT32 awe\_fwGetInputSampleRate\( AWEInstance \*pAWE,**

**UINT32 pinIdx**

**\);**

Returns the sample rate of an input pin.

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **pinIdx** which input pin

**void awe\_fwGetChannelCount\( AWEInstance \*pAWE,**

**UINT32 \*inCount,**

**UINT32 \*outcount**

**\);**

Returns the number of channels in the Layout's input and output pins.

\[in\] **pAWE** AWE instance pointer \(this\)

\[out\] **inCount** channels in primary input pin

\[out\] **outCount** channels in primary output pin

**INT32\* awe\_fwGetInputChannelPtr\( AWEInstance \*pAWE,**

**UINT32 pinIdx,**

**UINT32 chan,**

**INT32\* stride**

**\);**

Returns a pointer to where the platform should write input data to.

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **pinIdx** which input pin

\[in\] **chan** which channel

\[out\] **stride** data stride

**INT32\* awe\_fwGetOutputChannelPtr\( AWEInstance \*pAWE,**

**UINT32 pinIdx,**

**UINT32 chan,**

**INT32\* stride**

**\);**

Returns a pointer to location of processed output audio.

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **pinIdx** which input pin

\[in\] **chan** which channel

\[out\] **stride** data stride

**INT32 awe\_fwPump\( AWEInstance \*pAWE,**

**UINT32 layout\_no**

**\);**

Master framework pump function

Returns 0 on success or error code

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **layout\_no** the layout index to pump

**INT32 awe\_fwTick \( AWEInstance \*pAWE \);**

Call a module's set function if it has one.

Returns 0 on success or error code

\[in\] **pAWE** AWE instance pointer \(this\)

#### Required Variables

**size\_t g\_fastb\_heap\_index**

Index to next free heap word.

The heap is allocated from g\_fastb\_heap starting at zero.

**size\_t g\_master\_heap\_index**

Index to next free heap word.

The heap is allocated from g\_master\_heap starting at zero.

Index to next free heap word.

The heap is allocated from g\_slow\_heap starting at zero.

**DWORD g\_target\_control\_flags**

Set of target flags for various features.

Default is 0, and gives legacy operation.

## Tuning Interface

### Overview

_Tuning_ data from Audio Weaver Designer™ \(or third party tools\) arrives to the processor in packet form. For memory efficiency, the processing of these tuning commands is done in-place; meaning once each tuning command is processed, it is overwritten with a reply. The tuning interface processes commands and returns replies one at a time.

|  |  |
| :--- | :--- |
| INT32  | **awe\_fwPacketProcess** \(AWEInstance \* pAWE\) |
|  | Process a received binary Tuning packet. |
|  |  |
| void  | **awe\_fwPacketInit** \(DWORD \*packet\_buf, INT32 buf\_length\) |
|  | Initialize tuning communications. |
|  |  |
| INT32  | **awe\_fwPacketExecuteArray** \(AWEInstance \*pAWE, const UINT32 \*pCommands, UINT32 arraySize, UINT32 \*pPos\); |
|  | Executes tuning commands from an AWB file in local memory. |

|  |  |
| :--- | :--- |
| UINT32  | **packet\_buffer** \[\] |
|  | A user-space buffer to hold both incoming commands and generated replies |

### Detailed Description

#### Functions

**INT32 awe\_fwPacketInit \( AWEInstance \*pAWE,**

**INT32 buf\_length**

**\);**

Initialize packet communications.

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **buf\_length** Packet buffer length in DWords

**INT32 awe\_** **fwPacketProcess \( void \)**

Process a received Message

Returns E\_SUCCESS or E\_MESSAGE\_LENGTH\_TOO\_LONG, E\_CRC\_ERROR

**INT32 awe\_** **fwPacketExecuteArray\( UINT32\* pCommands,**

**UINT32 arraySize**

**\);**

Executes packet commands from an in memory array.

\[in\] **pCommands** Buffer with commands to execute

\[in\] **arraySize** Number of DWords in command buffer

## Control Interface

### Overview

The control interface is used in-product to allow HMI settings \(volume knob, e.g.\) and system-state \(battery level, e.g.\) to affect the audio processing. It’s also used to fetch data calculated in the Layout, such as RMS-level or event-detection flags.

![](.gitbook/assets/3%20%287%29.png)The API for the control interface is simple; firmware may read or write the state-structure of any module instance. To do this, the firmware first fetches a pointer to the module instance using **awe\_fwGetObjectByID\(\)**.

By default, ObjectIDs are dynamically assigned by the Runtime Core during run-time instantiation. Thus, if a designer wishes to statically assign a specific ObjectID to a specific module, they must do that in their Layout. To assign a static ID in the Layout, open the _Properties_ panel, select the _Build_ tab, and set the Object ID.

_Note: While the state/data of any module-instance may be accessed, it’s generally recommended that control data be exchanged using Source and Sink modules as ‘mailboxes’. This simplifies the firmware, reduces dependencies between the layout and the firmware, and allows control-data to be distributed and used throughout the Layout via control wires._

### Detailed Description

**INT32 awe\_** **fwGetObjectByID \( AWEInstance \*pAWE,**

**InstanceDescriptor\*\* pObject,**

**UINT32\* pClassID**

**\);**

Locate the object based on it’s objectID.

Returns E\_SUCCESS or E\_NO\_MORE\_OBJECTS

\[in\] **pAWE** AWE instance pointer \(this\)

\[in\] **ID** ID of object to find

\[out\] **pObject** pointer to receive found object pointer

\[out\] **pClassID** pointer to receive found object class

