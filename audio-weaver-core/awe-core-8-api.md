# AWE Core 8 API Quick Start



## Introduction

This document is meant to be a "Quick-Start" on how to integrate AWE Core. Advanced concepts are left out. Before reading this document, please see the [Theory Of Operation](http://download.dspconcepts.com/awecore/a00044.html) document.

The AWE Core™ is a hardware-independent, reconfigurable audio-processing engine. The AWE Core includes over 400 audio-processing Modules, from mixers and filters to compressors and FFTs, that can be combined to easily create complex signal processing flows. The AWE Core operates by passing audio through a user-defined sequence of these Modules, as directed by Configuration-data at run-time.

### Terms and Definitions

| Term | Definition |
| ---: | :--- |
| API | Application Interface |
| AWE Instance | The main AWE Core object |
| Tuning Interface | The interface by which commands/replies are transferred |
| Layout | Audio Weaver signal processing flow |
| Sub-Layout | Clock-divided section of a layout |
| AWB | Audio Weaver Binary File |
| AWD | Audio Weaver Design File |
| AWE | Audio Weaver |
| AWS | Audio Weaver Script File |
| BSP | Board Support Package |
| IPC | Inter Process Communication |
| RTOS | Realtime operating system |
| RT | Real-time |

### Additional Documentation

This document will go over the basics of integrating the AWE Core. It also includes a Doxygen-generated map of all the available API Functions, Macros, Data Structures, etc. [AWECore.h API Doc](http://download.dspconcepts.com/awecore/a00008.html).

For a brief view of the API, see the [AWE Core 8 Cheat Sheet](http://download.dspconcepts.com/AWECore8_cheatsheet.pdf).

For a more detailed description of the Tuning Protocol and the various transport methods, please see the Tuning Protocol document [here](http://download.dspconcepts.com/awecore/a00043.html).

## Overview of Integration Steps

Here are the basic steps to integrate AWE Core.

1. Include headers and link libraries
2. Define/allocate required variables
3. Configure the audio IO data structures
4. Initialize the API
5. Setup a tuning interface
6. Setup realtime audio
7. Setup a control interface \(optional based on product intent\)
8. Verify proper scheduling/priority.

## Include Headers and Link Libraries

### Required Header Files

* AWECore.h
  * This is the main API header file. It includes all of the API functions and structures required for an integration. See the API doc for details. [AWECore.h API Doc](http://download.dspconcepts.com/awecore/a00008.html).
* TargetProcessor.h, and the Target-specific header file
  * For instance, on an M7 DSPC will provide the file “ARM\_CortexM7\_TargetProcessor.h”.
* StandardDefs.h
  * Defines the AWE Core standard type definitions.
* Errors.h
  * When doing error checking on API calls, negative return values correspond to an error. The error can be determined by using the definitions in [Errors.h](http://download.dspconcepts.com/awecore/a00023.html).
* ModuleList.h
  * Every AWE Core deliverable contains a default ModuleList.h \(in the SampleApp/\[Target name\]/Source folder\) that includes a list of all the available modules. This file can be modified to account for custom modules, or modules that are not required for the target application.

### Required Libraries

* AWECore
  * This is the main AWE Core library.
  * The exact name and extension will vary from platform to platform, but will always have the basename "awecore".

### Helper Code / Optional

* AWECoreUtils \(AWEUtils.c/AWEUtils.h\)
  * AWECoreUtils contains various helper code including SPI/UART helper functions, a Multi-instance reply table generator, and a CRC computer for preparing custom replies to AWE Server. [AWECoreUtils.h API Doc](http://download.dspconcepts.com/awecore/a00011.html)
* ProxyIDs.h
  * ProxyIDs.h contains a list of all the AWE Core tuning commands. [ProxyIDs.h](http://download.dspconcepts.com/awecore/a00017.html)
* Additional Module Pack Libraries
  * Additional module pack libraries can be linked in \(VUI modules, 3rd party custom modules, etc\).

## Setting up the AWE Instance

### Declaring the Required Variables

A number of variables must be defined before configuring/initializing the AWE Instance.

* The AWEInstance object encapsulates a single AWE Core instance. Most API calls take an AWEInstance as the first argument. It must be defined so it doesn't go out of scope, for example as static or global, and be initialized to zeros.

  ```text
   //declare and initialize to zeros
   AWEInstance g_AWEInstance = {0};
  ```

Later, in the initialization section, it will become clear that there are several fields of the AWEInstance structure that must be configured before it can be used to process audio data.

* The IOPinDescriptor objects describe input and output data buffers and their properties. They must be defined so they don't go out of scope, for example as static.

  ```text
   static IOPinDescriptor s_InputPin;
   static IOPinDescriptor s_OutputPin;
  ```

* A module descriptor table that contains an array of pointers to modules that are available to the AWE Instance. The table is initialized at compile time. It must be defined so they don't go out of scope, for example as static.

  ```text
   //The list of class objects is defined in ModuleList.h
   const void* g_module_descriptor_table[] = { LISTOFCLASSOBJECTS};
  ```

* Memory heaps that the AWE Core uses for its own local processing. The integrator must provide word-aligned blocks of memory that are persistent. For example,

  ```text
   UINT32 g_FastHeapA[FAST_HEAP_A_SIZE];
   UINT32 g_FastHeapB[FAST_HEAP_B_SIZE];
   UINT32 g_SlowHeap[SLOW_HEAP_SIZE];
  ```

   where FAST\_HEAP\_A\_SIZE, FAST\_HEAP\_B\_SIZE, and SLOW\_HEAP\_SIZE are chosen appropriately for the system. See the [Optimization]() section for more details.

* A command packet buffer of type UINT32\[264\] that is the destination of tuning commands.

  ```text
   #define PACKET_BUFFER_SIZE 264
   UINT32 AWE_Packet_Buffer[PACKET_BUFFER_SIZE];
  ```

* A reply packet buffer of type UINT32\[264\] where the AWE Core places replies to tuning commands.

  ```text
   UINT32 AWE_Packet_Buffer_Reply[PACKET_BUFFER_SIZE];
  ```

**Note: The standard packet size is 264. If a different packet size is needed, please contact DSPC Engineering.**

### Configuring the AWE Instance

Now that the required variables have been declared, the AWE Instance will be configured by assigning its members and pointers.

* First, ensure that the AWE Instance is initialized to zeros.
* Assign the instanceID to 0.

  ```text
   g_AWEInstance.instanceId = 0;
  ```

* Assign the AWEInstance’s “pInputPin” and “pOutputPin” pointers to the input and output pin descriptors.

  ```text
   g_AWEInstance.pInputPin = &s_InputPin;
   g_AWEInstance.pOutputPin = &s_OutputPin;
  ```

* Assign the AWEInstance’s “pPacketBuffer” pointer to the command buffer that was declared above.

  ```text
   g_AWEInstance.pPacketBuffer = AWE_Packet_Buffer;
  ```

* Assign the AWEInstance’s “pReplyBuffer” pointer to the reply buffer. \(Note that this can be assigned to the same buffer as the send buffer if a single buffer for send/reply is desired.\)

  ```text
   g_AWEInstance.pReplyBuffer = AWE_Packet_Buffer_Reply;
  ```

* Assign the AWEInstance’s “packetBufferSize” member to 264.

  ```text
   #define PACKET_BUFFER_SIZE 264
   g_AWEInstance.packetBufferSize = PACKET_BUFFER_SIZE;
  ```

* Assign the “moduleDescriptorTable” pointer to the module descriptor table.

  ```text
   g_AWEInstance.pModuleDescriptorTable = g_module_descriptor_table;
  ```

* Assign the “numModules” member to the number of modules. This can be determined by taking the sizeof the module descriptor table and dividing it by the sizeof the first element of the module descriptor table.

  ```text
   UINT32 module_descriptor_table_size = sizeof(g_module_descriptor_table) / sizeof(g_module_descriptor_table[0]); 
   g_AWEInstance.numModules = module_descriptor_table_size;
  ```

* Assign the “numThreads” to the number of audio processing threads. See the [Multi-Rate]() section for more info.

  ```text
   g_AWEInstance.numThreads = 2; //dual threaded, supports two blocksizes
  ```

* Assign the “sampleRate” member to the fundamental sample rate of the system.

  ```text
   g_AWEInstance.sampleRate = 48000.0f;
  ```

* Assign the “fundamentalBlockSize” member to the fundamental block size of the system. This is typically the DMA block size. Every layout that is run on the system must have a block size that is a multiple of this fundamental block size.

  ```text
   #define AWE_BLOCK_SIZE 32
   g_AWEInstance.fundamentalBlockSize = AWE_BLOCK_SIZE;
  ```

* Assign the “pFlashFileSystem” member to 0 unless an Audio Weaver Flash File System will be implemented.

  ```text
   g_AWEInstance.pFlashFileSystem = 0;
  ```

* Assign the “fastHeapASize”, “slowHeapSize” and “fastHeapBSize” members to the heap sizes.

  ```text
   g_AWEInstance.fastHeapASize = FAST_HEAP_A_SIZE;
   g_AWEInstance.fastHeapBSize = FAST_HEAP_B_SIZE;
   g_AWEInstance.slowHeapSize = SLOW_HEAP_SIZE;
  ```

* Assign the “pFastHeapA”, “pSlowHeap” and “pFastHeapB” pointers to the heaps that were previously allocated.

  ```text
   g_AWEInstance.pFastHeapA = g_FastHeapA;
   g_AWEInstance.pFastHeapB = g_FastHeapB;
   g_AWEInstance.pSlowHeap = g_SlowHeap;
  ```

* Assign the “coreSpeed” and “profileSpeed” members to the CPU/profiling speed of the system. Only used for profiling displays in Audio Weaver tools.

  ```text
   g_AWEInstance.coreSpeed = 10e6f;
   g_AWEInstance.profileSpeed = 10e6f;
  ```

* Assign the “name”, this will be displayed in the AWE Server dialog as the name of the instance. Max length of 8 characters.

  ```text
   g_AWEInstance.pName = “mytarget”;
  ```

* Assign the "userVersion" member to be a UINT32 value. The value and meaning is entirely up to the integrator.

  ```text
   //this example represents a date
   g_AWEInstance.userVersion = (UINT32) 06212019;
  ```

### Initializing the IO Pins

The next step is to first initialize the IO Pins followed by initializing the AWE Instance using specific API functions.

#### Initializing the IO Pins

Simply call the awe\_initPin\(\) function and pass it the input pin, the desired channel count \(determined by audio HW\), and an optional pin name. See API doc for data type details.

```text
#define TWO_CHANNELS 2
int ret = awe_initPin(g_InputPin, TWO_CHANNELS, NULL);
```

The code above would initialize the input pin with 2 channels, and the default name.

Initializing the output pin is the same as the input pin, but the output pin is passed in.

```text
#define SIX_CHANNELS 6
int ret = awe_initPin(s_OutputPin, SIX_CHANNELS, "outpin");
```

The code above would initialize the output pin with 6 channels with the name "outpin".

### Initializing the AWE Instance

Next, initialize the AWE Instance by calling the awe\_init\(\) function. **This must happen as the last initialization step.** If awe\_init\(\) is called before the AWE Instance structure is configured or before the IO pins are initialized, the initialization will fail.

```text
int ret = awe_init(&g_AWEInstance);
```

The code above will initialize the AWE Instance.

## Setting up a Tuning Interface

The next step is to implement a _tuning interface_ – arguably the most important component of a successful AWE Core integration as it provides essential debugging capabilities. The tuning interface communicates commands/replies to and from the AWE Instance. For example, these commands can instantiate a layout, or set and query the value of a module parameter.

Different platforms will support different transport protocols for the tuning interface. AWE Server supports USB, TCP/IP \(sockets\), RS232, SPI, etc. Helper code is available to aid in the development of these different transport layers on the target. **It is the responsibility of the integrator to enable the transport protocol for the tuning commands to be passed to and from the platform.**

Here are the basic steps for setting up and using a tuning interface with AWE Server.

1. Setup/test the desired transport layer between the board and the PC. Make sure that packets of length 264 words can be passed back and forth \(echo\). The protocol can be verified completely independently of AWE Core using some kind of simple application.
2. Once data can be passed, the tuning interface should be setup to receive commands from AWE Server. This process varies from protocol to protocol.
3. Once commands are received from AWE Server, they will need to be processed by the AWE Instance. Once a command is received from AWE Server, copy it into the `AWE_Packet_Buffer` and then call `awe_packetProcess(&g_AWEInstance)` on the instance. Remember that this packet buffer has been registered with the AWE Instance, which is why the `awe_packetProcess(&g_AWEInstance)` function does not need to take an argument to the packet buffer.
4. The `awe_packetProcess()` function will process the packet, and then generate a reply in the `AWEInstance.pReplyBuffer`. If the same buffer is used for both send and reply, the reply message will overwrite the original command.
5. Finally, the reply is sent back to the AWE Server over the transport layer. See the following pseudocode representation of the complete transaction.

   ```text
   int sizeOfPacket = 264;
   readPacket(&AWE_Packet_Buffer, sizeOfPacket)
   awe_packetProcess(&g_AWEInstance)
   writePacket(&AWE_Packet_Buffer_Reply, sizeOfPacket)
   ```

## Real-time Audio

### RT Audio Introduction

The next step is to integrate real-time audio. AWE Core aside, real-time audio can be a tricky topic. Before attempting to integrate real-time audio into an AWE Core system, please ensure that the integrator has a basic understanding of digital audio and real-time audio processing. Here are some helpful links.

[http://www.rossbencina.com/code](http://www.rossbencina.com/code)

[Giulio Moro - Assessing the stability of audio code for real time low latency embedded processing](https://youtu.be/e1D5vCBWhdk)

### RealTime Audio Integration Steps

1. **Verifying Audio HW IO with a Passthrough** Ensure that audio can be cleanly passed between input/output without the AWE Core involved in the signal path. A simple sine wave input into the system can often be the best audio source for this type of test, as any distortion or dropouts are more easily audible than with a more complex signal. This step is critical to ensure a high fidelity audio system, and can vary greatly from platform to platform. For instance, an embedded target might utilize the HW’s audio IO DMA, while Linux systems may use ALSA or something similar.
2. **Determining Audio HW Settings and Configuring the AWE Instance.** These steps will describe how to set up the AWE Instance for RT Audio support.
   1. Configure the sample rate and block size. For example, if the audio HW reads in samples at a block size of 32, then the AWE Instance’s fundamental blockSize should be set to 32. The block size of any layout loaded on the target must be a multiple of this fundamental block size. If the HW’s sampling rate is 44100.0, then the AWE Instance’s sampleRate should be set to 44100.0.
   2. Determine the sample format of the audio HW. See the Audio Sample Formatting section below for more information on audio sample formatting.
   3. Determine the Input/Output channel count of the audio HW. AWE Core needs to be able to import/export data for each available HW channel. For each available hardware input channel, the application will need to call `awe_audioImportSamples()`, and, likewise, `awe_audioExportSamples()` will need to be called for each available hardware output channel. Using these channel-by-channel functions make it easy to aggregate multiple audio sources into a single, multi-channel input or output. Additionally, the Import and Export functions automatically handle what is referred to as 'channel matching'. Since AWE Core is designed to be flexible and Audio Weaver layouts are designed to be portable, the number of channels in a layout need not match the number of HW inputs and outputs in a system. If a layout has more I/O channels than the hardware channel count, then the Import and Export functions will automatically fill the input signal with additional zero'd channels, and will ignore any additional output channels. Similarly, if the layout loaded has fewer channels than the hardware channel count, then the Import and Export functions will ignore the additional HW input channels, and fill the additional HW output channels with zeros.
3. **The Realtime Audio Loop.** These steps describe what should happen inside the audio loop after configuration is complete.
   1. In the audio loop, check to see that a valid layout is loaded and that audio has been started with the API calls `awe_layoutIsValid()` and `awe_audioIsStarted()`.
   2. Call `awe_audioImportSamples()` for each input channel and pass in the audio data from the audio HW.
   3. Call `awe_audioExportSamples()` for each output channel and pass in the audio HW’s output data.
   4. Get the “layoutMask” from the awe\_audioGetPumpMask\(\) function. This will return a bit vector of the audio processing threads that need to be pumped. The number of audio threads to pump is determined by the number of clock-dividers \(bufferup/down paths\) in the layout. The layoutMask bit vector should be &’ed with 1,2,4,8 to determine the different threads. Each pump should happen on a different thread at different priorities. See the Multi-Rate section for more information.
   5. Call `awe_audioPump` for each available thread at the appropriate priority with the layout index determined by the layoutMask.
   6. Repeat.

### Audio Sample Formatting

The data type of the input and output audio data is determined by the audio hardware. Typically, digital audio is represented as fixed point data in 16, 24, or 32 bit depth. The audio sample formats supported by AWE Core's `awe_audioImportSamples()` and `awe_audioExportSamples()` functions, as defined in AWECore.h, are:

* Sample16bit - Fixed point 16 bit audio data stored in 16 bit data type. Also known as Q15 format.
* Sample24bit\_low - Fixed point 24 bit audio stored in 32 bit data type. Right justified, so the 24 bits of data are stored in the lowest 24 bits of the 32 bit word.
* Sample24bit\_high - Fixed point 24 bit audio stored in 32 bit data type. Left justified, so the 24 bits of data are stored in the highest 24 bits of the 32 bit word. If both right justified and left justified formatting is available on the hardware, then choose left justified \(Sample32bit\_high\) for a slight performance improvement.
* Sample32bit - Fixed point 32 bit audio data. Also known as Q31 format.

Internally, all inputs and outputs to Audio Weaver layouts are of type Sample32bit, also referred to as fract32 within a layout. This is done to guarantee portability of any signal processing layout to any target system, regardless of the hardware's native audio data type. The `awe_audioImportSamples()` and `awe_audioExportSamples()` functions will convert to and from the Sample32bit data as needed based on the integrator's supplied SampleType argument. If the target's native audio sample formatting is not one of those listed above, then the integrator will have to manually convert the data to one of the supported types before using the import and export functions.

Since some common target systems natively support floating point audio, helper functions are provided to convert between float&lt;–&gt;fract32 in AWECoreUtils.c, which is provided in the AWE Core package. Add AWECoreUtils.c to the project to access the sample-by-sample conversion functions `float_to_fract32` and `fract32_to_float`. See the API doc for AWECoreUtils.h for more info.

### Multi-Rate Processing

The AWE Core allows audio to be processed at multiple block rates. For example, consider a system with two processing paths: one that processes time-domain data in blocks of 32 samples and another that processes frequency-domain data in blocks of 64 samples but at 1/2 the rate of the time-domain path. Such a system is shown in the figure below. It uses BufferUp and BufferDown modules to connect the different block size domains. These modules effectively partition the layout into 2 sub-layouts operating at different block rates.

The two paths are executed at different rates and AWE Core’s `awe_audioGetPumpMask()` API call provides a mechanism to determine when the processing for each path should be initiated. Consider the following pseudocode:

```text
layoutMask= awe_audioGetPumpMask(&g_AWEInstance);

if (layoutMask & 0x1) raise(AWEProcess_HiPrio);
if (layoutMask & 0x2) raise(AWEProcess_LowPrio);

AWEProcess_HiPrio() 
{ 
    awe_audioPump(g_AWEInstance, 0); // small block size path
}
AWEProcess_LowPrio()
{ 
    awe_audioPump(g_AWEInstance, 1); // large block size path
}
```

This code tests which sub-layouts have accumulated enough data to execute. The layoutMask variable contains a 1 in each bit position corresponding to sub-layout that is ready to execute. For example, if layouts 0 and 1 are ready to execute,

```text
layoutMask = 0x00000003
```

Lower numbered sub-layouts correspond to smaller block sizes. In the pseudo-code, a signal is raised for each sub-layout that is ready to be pumped.

### Multi-Rate Scheduling

Real-time constraints dictate that if the 64 sample sub-layout is executed in the same context as the 32 sample sub-layout, both will need to complete processing before the next block of 32 samples arrives or real-time will be broken. This imposes an unnecessarily strict constraint on the 64 sample sub-layout – its processing need only be completed before the next block of 64 samples is accumulated. Thus, its processing time can, in principle, be spread over 2 of the 32 sample block times without breaking real-time. In practice, this is achieved by executing the two sub-layouts in separate contexts. The sub-layout with the shorter block size should have higher priority so that it can preempt the processing of the sub-layout with the longer block size. In this way, the real time constraints of each sub-layout can be accommodated. The following figure shows the timing for this example:

## Deferred Processing

There are certain AWE modules that need to perform time consuming calculations, for example when the cutoff frequency of a Second Order Filter module is changed, the filter coefficients need to be recalculated. Performing these calculations in the audio processing context can cause it to overrun. To address this issue, certain modules defer those calculations until the firmware calls `awe_deferredSetCall()`.

An integrator can check for any required deferred processing using the return value of `awe_audioPump()`, which will return `TRUE` if any deferred processing is pending. If deferred processing is needed, then call `awe_deferredSetCall()` at a priority that is lower than the audio processing. `awe_deferredSetCall()` performs deferred processing for a single module, and returns `TRUE` if there is more deferred processing that is pending. Thus it should be called repeatedly until it returns `FALSE`.

```text
//g_bDeferredProcessingRequired is returned by awe_audioPump()

if (g_bDeferredProcessingRequired || bMoreProcessingRequired)
{
    g_bDeferredProcessingRequired = FALSE;
    bMoreProcessingRequired = awe_deferredSetCall(&g_AWEInstance);
}
```

## Standalone Operation

At this point, the integrator should be able to load a layout from Audio Weaver Designer and run it on the target via the Tuning Interface. Once a Layout in Designer has been completed, it is easy to switch to stand-alone operation. Simply ask Audio Weaver to "Generate Target Files", and the Layout's configuration-data will be generated as a C array to be compiled into the system.

### Loading from an Array

From the viewpoint of AWE Core, the signal processing layout is described by a data array of binary Audio Weaver commands. This command array is generated by Audio Weaver Designer using the Tools/Generate Target Files menu item and enabling the \[BaseName\]\_initAWB checkbox. The layout is loaded into an AWE Core instance by making a call to awe\_loadAWBfromArray with the command array, its size in 32-bit words, and the AWEInstance as arguments. If an error occurs during loading, then the offset of the offending AWB command is returned to the pPos argument.

```text
INT32 err = awe_loadAWBfromArray (pAWE, pCommands,arraySize, &nErrOffset );
if (err)
{
    // report the error
    printf(“error code %u due to command at position %u\n”, err, nErrOffset);
    // handle the error
    ...
}
```

`awe_loadAWBfromArray()` will load the entire array of commands and process them locally on the AWE Instance. If an array of commands needs to be loaded on a remote instance, it can be loaded command by command with the `awe_getNextAWBCmd()` helper function in AWECoreUtils. Each command is parsed so that they can be routed to the remote instance.

## AWE Core Scheduling and Priority

All priority and interrupt issues are managed outside of the AWE Core by the firmware integrator since the AWE Core has no knowledge of the supporting processing environment it is being integrated into. The following processing context may be implemented using interrupt handlers or using OS threads/tasks. The main requirement is that the processing context is implemented at different preemptible priority levels.

A basic Audio Weaver platform has a minimum of five priority levels of processing. From highest priority to lowest priority:

* **Priority 1: Tuning Communication I/O** **Task:** Receives/sends data over the communication transport layer, for example RS232, USB, SPI, or Ethernet. _Note: Tuning commands are not processed in this context; data is simply moved into and out of the tuning packet buffer._ **Action**: Insert received data into the tuning packet buffer or send reply data from tuning packet buffer. When a complete packet has been received, signal the low priority idle task to process it.
* **Priority 2: Audio I/O** **Task:** Complete audio frame has been received. Blocks of audio samples are exchanged between DMA buffers and the AWE Core input and output buffers. **Action:** Copy new audio samples into AWE layout input buffers copy latest AWE layout processed audio samples to output buffers if enough audio samples are available trigger Priority 3 and/or Priority 4 tasks to perform signal processing.
* **Priority 3: Audio Processing** **Task:** Process one AWE layout-defined frame of audio samples. **Action:** At this priority level, the active signal processing layout is processed with `awe_audioPump()`.
* **Priority 4: Lower Priority Audio Processing \(multi-rate\)** **Task:** Process one larger AWE layout-defined frame of audio samples. **Action:** At this priority level, longer duration signal processing is initiated with `awe_audioPump()` but at a lower priority than the normal audio processing above.
* **Priority 5: Background Processing** **Task:** In background / idle loop context, non-real-time tasks are processed. This includes the processing of tuning commands, deferred processing, and any Control Interface processing. **Action:**
  * Process a received tuning command.
  * Process any deferred processing needed for a single module.
  * Perform any needed interaction with the layout from the firmware.

These three actions **must be atomic** to avoid the possibility of race conditions due to concurrent access of resources.

## Setting up a Control Interface

### Control Interface Overview

A control interface lets the system interact with modules in the running layout directly from the firmware. The following API calls define the functionality of the control interface:

Get/set a value of a module:

```text
awe_ctrlGetValue()
awe_ctrlSetValue()
```

 Get/Set the status of a module \(bypassed, active, inactive, muted\):

```text
awe_ctrlSetStatus()
awe_ctrlGetStatus()
```

Check if a module exists, and if so return its ClassID:

```text
awe_ctrlGetModuleClass()
```

The following functions provide finer grained control over how module variables get set and are for advanced users:

```text
awe_ctrlSetValueMask()
awe_ctrlGetValueMask()
```

### Control Interface Steps

To access a module and control it via the Control Interface,

* Create the desired layout. In the build tab of a module, assign an objectID to the module's that will be accessed. The objectID must be a value between 30000-32767.
* Generate a ControlInterface.h header file from the layout with Tools-&gt;Generate Target Files. Make sure the \[BaseName\]\_ControlInterface.h box is checked.
* Include the generated \[BaseName\]\_ControlInterface.h file in the AWE Core integration. This file will contain all of the define's required to use the API's control functions.
* Check if the module exists and is of the right class with `awe_ctrlGetModuleClass()`.
* Use one of the awe\_ctrl functions to set/get something about a module.
  * The `handle, length` arguments will all be defined in the \[BaseName\]\_ControlInterface.h file that was generated. See the API doc for details about the control functions arguments/return values.
* Check the return value for errors and handle appropriately.

See the following example.

```text
    // Does the current AWE model have a SinkInt module with this control object ID?
    if (awe_ctrlGetModuleClass(&g_AWEInstance, AWE_SinkInt1_value_HANDLE, &classID) == OBJECT_FOUND)
    {
        // Check that module assigned this object ID is of module class SinkInt
        if (classID == AWE_SinkInt1_classID)
        {
            // SinkInt module (value is an array)
            awe_ctrlGetValue(&g_AWEInstance, AWE_SinkInt1_value_HANDLE, &nValue, 0, 1);
        }
    }
```

## Optimization

The AWE Core can be optimized for a system for heap space and for image size.

### Heap Sizing and Placement

In a typical workflow, the integrator would decide on heap size/placement for development and finally optimized for production.

To decide on heap size/placement for board bring-up, set all three heaps to some small number of 1K word blocks say \(1024 \* 5\). When the BSP is fully developed inspect the memory map to determine how much free space is available in the memory sections that have been assigned to the heaps. Then adjust the heap space to use as much of this memory as is practical. A typical assignment of heap might be to assign processor “tightly coupled memory” to fast heap A, processor RAM to fast heap B, and off-chip SDRAM to the slow heap.

Audio Weaver Designer can also generate the absolute smallest heap sizes needed for a specific layout via Tools-&gt;Generate Target Files and selecting \[BaseName\]\_HeapSize.h. This can greatly reduce the memory footprint and aid in optimizing an AWE Core integration.

### Module List Optimization

The ModuleList.h file that is delivered with AWE Core contains a large set of modules. This is convenient during development to provide a large selection of modules with which to design layouts. To optimize for a specific layout at production time, Audio Weaver Designer can generate a ModuleList.h with only the modules used by that layout using Tools-&gt;Generate Target Files-&gt;\[BaseName\]\_ModuleList.h. Since most modern linkers will only link those modules referenced, this will significantly reduce the image size.

## Multi Instance

### Multi Instance Introduction

AWE Core target systems can contain multiple AWE Instances. This is useful on a platform with multiple cores when a system needs to do signal processing on each core.

Before setting up a multi instance tuning interface, we recommend implementing a single instance tuning interface. This will solidify the theory of operations and allow for easier understanding of the multi instance model.

### Multi Instance Concepts/Use Cases

AWE Core packets always contain a prefixed address called an “instanceID”. On a single instance system, this instanceID is always 0. However, when developing a multi instance system, commands can be addressed to different instanceID’s.

When using Designer, the instanceID of an AWE command is determined by which instance is selected in the dropdown window of Designer.

It is important to note that there can only be one tuning interface between Server and the system. That single tuning interface will receive the packets for all instances and the BSP integrator must route them to the correct instance.

### Multi Instance Tuning Interface Setup

#### Basic Overview of Multi Instance Integration Steps

Integrating a multi-instance AWE Core system can be broken into the following basic steps. This assumes that the integrator is able to implement a single instance tuning interface.

1. Setup and initialize the multiple AWE Instances with appropriate instanceID’s.
2. Setup a tuning interface and tell Server how many instances there are. Do this by generating a reply to the GetInstanceTable AWE command sent from AWE Server.
3. Route the packets to the correct instance based on the packet’s instanceID prefix.
4. Send the replies back to Server.

#### Detailed Multi Instance Integration Steps

1. Allocate/initialize the multiple AWE Instances with their instanceIDs. The instanceID is stored in the high order 4 bits of an 8 bit word. Therefore the first instance must always be 0, and the following cores will increment by 16. **IMPORTANT: InstanceID’s should always increment by 16. So the instanceID’s for a 2 instance system would be 0, 16. For a 3 instance system, 0, 16, 32. Etc...**
2. Declare an instance table array as a UINT32\[numInstances\], where the elements are the instanceID’s of the AWE instance’s that were just initialized.

   ```text
   //the following instance table represents a two-instance system.
   UINT32 numInstances = 2;
   UINT32 instanceIDs[numInstances] = { 0, 16 };
   ```

3. Implement a tuning interface to pass AWE commands between AWE Server and the target \(See the Single Instance Tuning Interface Section\).
4. When multi instance is to be supported, the BSP author will setup the tuning interface to listen for the AWE Command “PFID\_GetInstanceTable” \(use the PACKET\_OPCODE macro and the enum in ProxyIDs.h to determine this command\). When this command is received, use the GenerateInstanceTableReply function in AWECoreUtils to generate a reply to the PFID\_GetInstanceTable command. Send the instance table reply back to the Server over the tuning interface. Now AWE Server knows how many instances there are in the system, and can create a dropdown list of the instances in AWE Server and Designer.

```text
if (opcode == PFID_GetInstanceTable)
{
    GenerateInstanceTableReply(AWE_Packet_Buffer_Reply, numInstances, instanceIDs);
    writePacketToServer(AWE_Packet_Buffer_Reply);
}
```

1. Implement a packet router. The tuning interface must be able to send/receive packets from all instances. The packet router will take a packet from the Server, strip off its instanceID \(use the macro PACKET\_INSTANCEID\), and then route the command to the appropriate instance.

### Multi Instance Pseudocode Examples

The following pseudocode example is for a chip with two signal processing cores with the first core providing the tuning interface.

```text
TuningInterface()
{
    AWE_Packet_Buffer = receivePacketFromServer();

    if (PACKET_OPCODE(AWE_PACKET_BUFFER)) == PFID_GetInstanceTable)
    {
        *AWE_Packet_Buffer = *GenerateInstanceTableReply(AWE_Packet_Buffer, numInstances, instanceIDs);
        writePacketToServer(AWE_Packet_Buffer);
    }
    else if (PACKET_INSTANCEID(AWE_Packet_Buffer) == 0)
    {
        awe_packetProcess(AWEInstance0);
        writeReplyToServer();
    }

    else if (PACKET_INSTANCEID(AWE_Packet_Buffer) == 16)
    {
        sendToInstance16()
        //command is processed on instance 16
        readReplyFromInstance16()
        writeReplyToServer();
    }
}
```

Here is a pseudocode example of a multi-instance tuning interface that is implemented on an MCU that has no knowledge of the AWE Core instance’s. The system has two AWE instances on separate processors, with ID’s of 0 and 16.

```text
TuningInterface()
{
    Tuning_PacketBuffer= receivePacketFromServer()

    if (PACKET_OPCODE(Tuning_PacketBuffer)) == PFID_GetInstanceTable)
    {
        GenerateInstanceTableReply(Tuning_PacketBuffer, numInstances, instanceIDs);
        writePacketToServer(Tuning_PacketBuffer);
    }
    else if (PACKET_INSTANCEID(Tuning_PacketBuffer) == 0)
    {
        sendPacketToInstance0(Tuning_PacketBuffer)
        //Instance 1 gets packet and calls awe_packetProcess
        receiveReplyPacketFromInstance0()
        writeReplyToServer()
    }

    else if (PACKET_INSTANCEID(AWE_Packet_Buffer) == 16)
    {
        sendPacketToInstance16()
        //Instance 16 gets packet and calls awe_packetProcess
        receivePacketFromInstance16()
        writeReplyToServer()
    }
}
```

## Troubleshooting and Common Pitfalls

There are a few common pitfalls when BSP authors are integrating AWE Core. The most common problems are scheduling issues, specifically when the audio thread is not at a high enough priority and is preempted by the packet or deferred processing. RT audio needs to be running at a very high priority, just below the Tuning Interface IO thread.

