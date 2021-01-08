# Theory of Operation

## Introduction

The AWECore is an embedded library enabling Audio Weaver signal processing and remote tuning. This document describes the theory of operation and the basic concepts of Audio Weaver.

The AWECore deliverable consists of a set of libraries to be integrated into embedded firmware. It is context-agnostic, so it can be used with an OS or a simple priority-based scheduling scheme.

For a detailed API integration guide with code examples, see the AWECore 8 API Quick Start.

## What is AWE Core?

AWE Core is a data-driven engine. It contains a set of signal processing modules. The signal processing logic, which we refer to as a ‘layout’, is constructed at runtime using a set of commands that configure and connect some subset of these modules. Commands add modules to the active layout, connect wires between modules, configure module parameters, and start/stop audio processing.

**Note: a signal processing Layout is completely defined by a set of tuning commands**.

We refer to this process as dynamic instantiation. The following is a set of tuning commands that dynamically create a signal processing layout...

```text
0,create_wire,wire1,48000,2,32,0,32,0,4,0,0
0,create_wire,wire2,48000,2,32,0,32,0,4,0,0
0,create_wire,wire3,48000,2,32,0,32,0,4,0,0
0,bind_wire,wire1,Input
0,bind_wire,wire2,Output
0,create_module,SYS_toFloat,ModuleTypeConversion,1,1,0,wire1,wire3,1,0
0,create_module,SYS_toFract,ModuleTypeConversion,1,1,0,wire3,wire2,0,1
```

During signal processing development these commands are created by Audio Weaver Designer™. Matlab scripts, 3rd party tools, or other scripts such as Python.

For a product to run stand-alone, the complete set of commands that defines a Layout \(both in topology and tuning\) is exported by AWE Designer as an “Audio Weaver Binary” \(.AWB\) file or a C-array with the commands listed in ASCII hex. These commands can then be stored in non-volatile memory or compiled in as a C-array.

## Data Interfaces

AWECore exchanges three types of data; Audio, Tuning, and Control.

**Audio data** Audio data is processed by the AWE Core signal processing component.

**Tuning data** is what dynamically configures the signal-processing, including the topology of the modules and the tuning parameters for each, such as filter coefficients or channel gain. This data is in the form of commands, typically issued by Audio Weaver Designer™ \(or 3rd party tools / scripts\).

**Control data** consists of parameters exchanged between firmware components and the Audio Weaver signal processing logic.

## Tuning Model

A signal-processing Layout is instantiated in the AWE Core through a set of commands. From the perspective of the integrator and system designer, there are two scenarios to understand - based on the origin of the commands.

**Design-Time:** First, while a new product is being developed, DSP Engineers, Acoustic Engineers, and System Engineers will use PC-based tools like AWE Designer to create and refine the signal processing Layout. During this phase, tuning commands are sent to the AWE Core from the PC in realtime.

**Standalone:** Later, when developers are done tweaking and tuning a product’s signal-processing Layout, the commands that instantiate it may be compiled in as a C array, or written to non-volatile memory.

### Design-Time Operation

When using AWE Designer to create and tune a signal processing layout, tuning commands \(and the resulting replies\) are exchanged between the PC and the AWE Core running on the firmware. The full path that these tuning commands travel is shown below.

Communication between Designer and Server uses plain text commands. AWE Server then converts each text command into a binary packet and sends it to the target platform over the specified tuning transport \(e.g. Ethernet, USB, RS232, etc.\). Scripts containing the text commands are called Audio Weaver Scripts \(AWS\). Scripts containing the binary commands are called Audio Weaver Binary \(AWB\) scripts.

For a more detailed description of our Tuning Protocol and the AWB command structure, please see the Tuning Protocol document [here](http://download.dspconcepts.com/awecore/a00043.html)

A tuning interface must be developed for the firmware side to handle the chosen tuning transport/peripheral. This communication subsystem may respond to interrupts from an on-chip peripheral, such as UART or SPI, or it may process packets received via USB, network, or shared-memory. The tuning communication subsystem is simply a data-handler, exchanging messages between the tuning transport and the AWE Core. In general it has to run at a fairly high priority level in order not to miss communication events. However, the actual command handling must be done in a deferred procedure call at a lower priority than the signal processing.

Note that this command protocol is strictly initiated by the host \(typically AWE Server\). The host issues a command and will not issue another command until the original command is complete or times out. The firmware can never issue a command. It can only respond with a single reply to the current command.

### Standalone Operation

Once configuration and tuning is complete, a Layout can be saved on the product to allow stand alone operation. To do this, the Layout is exported as an Audio Weaver Binary \(.AWB\) file or as a C Array \(C file and Header file\). In AWE Designer, select Tools / Generate Target Files. AWB’s may be stored in-product in two ways: either ‘compiled-in’, where the AWB is embedded as static data in the processor’s executable, or as a separate data-object stored in non volatile memory.

#### Compiled-in AWB

The simplest way to store and load an AWB is to use Audio Weaver Designer to generate a C array that will be compiled in to the target. The array is then loaded \(i.e. executed\) immediately after booting, and audio processing starts immediately with no external activity. When integrating the AWE Core into an application, compiling-in the AWB is the recommended first step towards standalone operation. This is pictured below.

Audio Weaver Designer will automatically generate this pre-initialized array in a .C file by selecting Tools -&gt; Generate Target Files.

To load the Layout this AWB file represents, simply call the following function during post-boot init.

```text
INT32 awe_loadAWBfromArray(AWEInstance *pAWE, const UINT32 *pCommands, UINT32 arraySize, UINT32 *pPos);
```

### Control Model

API functions are provided to allow the firmware to directly interact with modules by setting/getting their variables and execution status. Collectively these API functions are referred to as the Control Interface.

### Audio-Processing Model

#### Block Processing

As audio samples are received, they are buffered into blocks. These blocks of audio samples are then copied into the AWECore subsystem, where they are processed.

A fundamental concept for AWECore is that the layout signal processing block size can differ from the DMA block size. For example, the firmware may be configured for a DMA block size of 32 samples, while a signal processing layout may use a block size of 64, 128, 256, etc. The one limitation is that the layout block size must be an integer multiple of the DMA block size. This decoupling allows the DSP engineer to easily make tradeoffs between MIPs, memory, and latency without having to modify or rebuild the target firmware. As shown in Figure 13, the AWE Core subsystem has internal buffering to automatically handle this decoupling. In the example in Figure 3, the signal processing layout operates on blocks 4x the DMA block-size.

The following steps illustrate the audio processing.

1. When an audio frame is received it is copied from the DMA receive buffers to the input buffers of the AWE Core layout.
2. Likewise, since input and output are synchronous, newly processed audio is copied from the AWE Core layout output buffers to the hardware output buffers.
3. When the number of audio frames accumulated matches the number of audio frames needed, the layout is processed.

