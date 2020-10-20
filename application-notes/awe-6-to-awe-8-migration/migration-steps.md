---
description: >-
  The following sections highlight the differences between AWE 6 and AWE 8
  API’s.
---

# Migration Steps

The following sections are broken into implementation sections to highlight the differences between AWE 6 and AWE 8 API’s. For brevity, simple use cases are compared and no error handling is shown.

## AWECore 6 -&gt; AWECore 8

### Initialization

Both AWE6 and AWE8 share the idea of an AWEInstance structure, which is initialized with the user’s configuration and passed to almost all AWE Core API functions. There are some key differences between AWE 6 and AWE8 AWEInstances and how they are initialized, which are discussed below.

#### **AWE Core 6**

1. Define the required global variables, AWEInstance, IOPinDescriptors, ModuleDescriptorTable, CoreDescriptor etc.

```c
AWEInstance g_AWEInstance;
CoreDescriptor g_Core[1];
 
UINT32 g_master_heap_size = MASTER_HEAP_SIZE;
UINT32 g_slow_heap_size = SLOW_HEAP_SIZE;
UINT32 g_fastb_heap_size = FASTB_HEAP_SIZE;
UINT32 g_master_heap[MASTER_HEAP_SIZE];
UINT32 g_fastb_heap[FASTB_HEAP_SIZE];
UINT32 g_slow_heap[SLOW_HEAP_SIZE];
 
const ModClassModule *g_module_descriptor_table[] =
{
     LISTOFCLASSOBJECTS
};
 
static IOPinDescriptor s_InputPin[1] = { 0 };
static IOPinDescriptor s_OutputPin[1] = { 0 };

```

2. Memset the AWEInstance to 0’s, and then configure required struct members \(assign IOPinDescriptors, ModuleDescriptorTable, callbacks, m\_coreID, heaps, heap sizes\)

```c
memset(&g_AWEInstance, 0, sizeof(AWEInstance) );

g_AWEInstance.m_pAwe_pltAudioStart = awe_pltAudioStart;
g_AWEInstance.m_pAwe_pltAudioStop = awe_pltAudioStop;

g_AWEInstance.m_pInterleavedInputPin = s_InputPin;
g_AWEInstance.m_pInterleavedOutputPin = s_OutputPin;

g_AWEInstance.m_module_descriptor_table_size = g_module_descriptor_table_size;
g_AWEInstance.m_pModule_descriptor_table = g_module_descriptor_table;

g_AWEInstance.m_coreID = CORE_ID;

g_AWEInstance.m_master_heap_size = MASTER_HEAP_SIZE;
g_AWEInstance.m_slow_heap_size = SLOW_HEAP_SIZE;
g_AWEInstance.m_fastb_heap_size = FASTB_HEAP_SIZE;

g_AWEInstance.m_master_heap = g_master_heap;
g_AWEInstance.m_slow_heap = g_slow_heap;
g_AWEInstance.m_fastb_heap = g_fastb_heap;

```

3. Initialize AWEInstance with desired targetInfo members via awe\_fwInitTargetInfo. Pass AWEInstance and other 19 targetInfo arguments.

```c
awe_fwInitTargetInfo(&g_AWEInstance,
                        CORE_ID,
                        CORE_SPEED,
                        SAMPLE_SPEED,
                        "ST32F407",
                        PROCESSOR_TYPE_CORTEXM4,
                        HAS_FLOAT_SUPPORT,
                        HAS_FLASH_FILESYSTEM,
                        NO_HW_INPUT_PINS,
                        NO_HW_OUTPUT_PINS,
                        IS_SMP,
                        NO_THREADS_SUPPORTED,
                        FIXED_SAMPLE_RATE,
                        INPUT_CHANNEL_COUNT,
                        OUTPUT_CHANNEL_COUNT,
                        VER_DAY, VER_MONTH, VER_YEAR,
                        AWE_FRAME_SIZE,
                        MAX_COMMAND_BUFFER_LEN
                        );

```

4. Initialize the IOPinDescriptors and then configure them with awe\_SetPackedName

```c
s_InputPin[0].sampleRate = FIXED_SAMPLE_RATE;
s_InputPin[0].wireInfo1 = nInputWireInfo;
s_InputPin[0].wireInfo3 |= CLOCK_MASTER_BIT;
awe_SetPackedName(s_InputPin[0].m_pinName, "Input");
 
s_OutputPin[0].sampleRate = FIXED_SAMPLE_RATE;
s_OutputPin[0].wireInfo1 = nOutputWireInfo;
awe_SetPackedName(s_OutputPin[0].m_pinName, "Output");

```

5. Call awe\_fwInit\(\) on the AWEInstance.

```c
awe_fwInit(&g_AWEInstance); 
```

6. Assign the CoreDescriptor.m\_pSendCommand callback to a user defined packet processing callback, and CoreDescriptor.m\_coreID to the same coreID as the AWEInstance.

```c
g_Core[0].m_pSendCommand = awe_pltCoreSendCommand;
g_Core[0].m_coreID = CORE_ID;
```

7. Call awe\_pltDefineCores with the CoreDescriptor variable

```c
awe_pltDefineCores( (CoreDescriptor *)g_Core, NUM_CORES);
```

#### **AWE Core 8**

1. Define the required global variables \(AWEInstance, IOPinDesctiptors ModuleDesciptorTable/size\)

```c
AWEInstance g_AWEInstance;
 
static IOPinDescriptor s_InputPin = { 0 };
static IOPinDescriptor s_OutputPin = { 0 };
UINT32 g_fastA_heap[FASTA_HEAP_SIZE];
UINT32 g_fastB_heap[FASTB_HEAP_SIZE];
UINT32 g_slow_heap[SLOW_HEAP_SIZE];
 
const void *g_module_descriptor_table[] =
{
    (void *)LISTOFCLASSOBJECTS
};
 
AWE_MOD_SLOW_DM_DATA UINT32 g_module_descriptor_table_size = sizeof(g_module_descriptor_table) / sizeof(g_module_descriptor_table[0]);
```

2. Initialize the IOPinDesctiptors with awe\_initPin\(\).

```c
awe_initPin(&s_InputPin, INPUT_CHANNEL_COUNT, NULL);
awe_initPin(&s_OutputPin, OUTPUT_CHANNEL_COUNT, NULL);
```

3. Memset the AWEInstance to 0’s, and configure by assigning required members \(target info, pins, heaps, instanceID, etc\).

```c
memset(&g_AWEInstance, 0, sizeof(AWEInstance) );
g_AWEInstance.userVersion = USER_VER;
g_AWEInstance.instanceId = INSTANCE_ID;
g_AWEInstance.coreSpeed = CORE_SPEED;
g_AWEInstance.profileSpeed = CORE_SPEED;
g_AWEInstance.pName = "ST32F407";
g_AWEInstance.numThreads = NUM_AUDIO_THREADS;
g_AWEInstance.pModuleDescriptorTable = g_module_descriptor_table;
g_AWEInstance.numModules = g_module_descriptor_table_size;
g_AWEInstance.sampleRate = AUDIO_SAMPLE_RATE;
g_AWEInstance.fundamentalBlockSize = AUDIO_BLOCK_SIZE;
g_AWEInstance.fastHeapASize = FASTA_HEAP_SIZE;
g_AWEInstance.fastHeapBSize = FASTB_HEAP_SIZE;
g_AWEInstance.slowHeapSize = SLOW_HEAP_SIZE;
g_AWEInstance.pFastHeapA = g_fastA_heap;
g_AWEInstance.pFastHeapB = g_fastB_heap;
g_AWEInstance.pSlowHeap = g_slow_heap;
g_AWEInstance.pPacketBuffer = g_packet_buffer;
g_AWEInstance.packetBufferSize = MAX_COMMAND_BUFFER_LEN;
g_AWEInstance.pReplyBuffer = g_packet_buffer;
```

4. Initialize the AWEInstance with `awe_init()`.

```c
awe_init(&g_AWEInstance);
```

**Initialization Summary**

**AWEInstance** **Initialization**

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
void awe_fwInit(AWEInstance * pAWE);

void awe_fwInitTargetInfo(AWEInstance *pAwe, UINT32 coreID, float coreClockSpeed, float sampleSpeed, const char *name, INT32 procType, INT32 isFloat, INT32 isFlash,INT32 nInputs, INT32 nOutputs, INT32 isSMP, INT32 threads, float sampleRate, INT32 nInputChannels, INT32 nOutputChannels, INT32 nVerDay, INT32 nVerMonth, INT32 nVerYear, UINT32 blockSize, UINT32 commandBufferLen);

void awe_fwPacketInit(UINT32 *packet_buf, UINT32 buf_length);

void awe_pltPacketInit(UINT32 * packet_buf, INT32 buf_length);

void awe_pltDefineCores(CoreDescriptor * pCores, INT32 nCores);
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_init(AWEInstance *pAWE);
// All targetinfo/instance config happens to AWEInstance variable before 
// calling awe_init() assumes buffer ptr’s assigned to AWEInstance packet/reply 
// buffer members, and instanceID member set appropriately (no need for 
// CoreDescriptor or awe_pltDefineCores)
```
{% endtab %}
{% endtabs %}

**IO** **Pin** **Initialization**

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
void awe_fwInit_io_pins(AWEInstance *pAWE, UINT32 clock_domain);

#define INFO1_PROPS(channels, maxBlockSize, complex, nSizeBytes) \ ((channels & 0x3ffU) | ((maxBlockSize & 0x1ffffU) << 10) | ((complex & 0x1U) << 27) | ((nSizeBytes & 0xfU) << 28))

void awe_SetPackedName(UINT32 *pPackedName, const char *name);
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_initPin(IOPinDescriptor *pPin, UINT32 channels, const char *name);
```
{% endtab %}
{% endtabs %}



[AWE Core 6 vs AWE Core 8 Detailed Discussion](https://docs.google.com/document/d/1SMeZVkrGHtXNMz3L3qZ25NDOJqOKfEr18Z_7t3aJw3E/edit?usp=sharing)



### Loading Layouts

Both AWE Core 6 and AWE Core 8 support loading AWB layouts stored as C arrays. These C arrays are generated from AWE Designer via the Tools-&gt;Generate Target Files menu. These C arrays are then compiled directly into the target application, and the API’s shown below are used to load the AWB.

**Layout API Summary**

AWE Core 6 and AWE Core 8 provide very similar functions to load query information about the loaded layout, listed below.

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
INT32 awe_fwPacketExecuteArray(const UINT32 *pCommands, UINT32 arraySize, UINT32 *pPos);

void awe_fwGetChannelCount(const AWEInstance *pAWE, UINT32 pinIdx, UINT32 *inCount, UINT32 *outCount);

UINT32 awe_fwGetInputBlockSize(const AWEInstance *pAWE, UINT32 pinIdx);

FLOAT32 awe_fwGetInputSampleRate(const AWEInstance *pAWE, UINT32 pinIdx);

N/A. User manually checks the layout head.
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_loadAWBfromArray(AWEInstance *pAWE, const UINT32 *pCommands, UINT32 arraySize, UINT32 *pPos);

void awe_layoutGetChannelCount(const AWEInstance *pAWE, UINT32 pinIdx, UINT32 *inCount, UINT32 *outCount);

INT32 awe_layoutGetInputBlockSize(const AWEInstance *pAWE, UINT32 pinIdx, UINT32 *blockSize);

INT32 awe_layoutGetInputSampleRate(const AWEInstance *pAWE, UINT32 pinIdx, FLOAT32 *sampleRate);

INT32 awe_layoutIsValid(const AWEInstance *pAWE);
```
{% endtab %}
{% endtabs %}

#### 

### Tuning Interface

The main distinction in the tuning interface between AWE Core 6 and AWE Core 8 is that AWE Core 6 considers an instance to have two main components, the “Framework”, and the “Platform”. Therefore, AWE 6 has two separate API’s to process packets, awe\_fwPacketProcess, awe\_pltTuningTick. Both of these API’s must be invoked in the correct context to successfully open a tuning interface and connect to AWE Server running on the PC.

AWE Core 8 does not have this notion of Platform and Framework components, so all tuning commands are handled with a single API awe\_packetProcess\(\).

#### **AWE Core 6**

1. User defines a packet buffer and initializes it with awe\_fwPacketInit, and awe\_PltPacketInit.

```cpp
UINT32 s_PacketBuffer[MAX_COMMAND_BUFFER_LEN] = {0};
awe_fwPacketInit(s_PacketBuffer, MAX_COMMAND_BUFFER_LEN);
awe_pltPacketInit(s_PacketBuffer, MAX_COMMAND_BUFFER_LEN);
```

2. User defines a callback function that invokes awe\_fwPacketProcess\(\) function, and assigns the callback’s function ptr to the CoreDescriptor.m\_pSendCommand member \(see Init section\). User must ensure that the CoreDescriptor.m\_coreID member matches the AWEInstance.m\_coreID member \(coreID 0 for single core systems\)

```cpp
g_Core[0].m_pSendCommand = awe_pltCoreSendCommand;
g_Core[0].m_coreID = CORE_ID;
INT32 awe_pltCoreSendCommand(void * pData, UINT32 * txBuffer, UINT32 * rxBuffer)
{
 AWE_UNUSED_VARIABLE(pData);
 AWE_UNUSED_VARIABLE(txBuffer);
 AWE_UNUSED_VARIABLE(rxBuffer);
 return awe_fwPacketProcess(&g_AWEInstance);
}
```

3. A user sets up some communication layer code \(unrelated to AWE, usually USB, RS232, TCP,etc\) to receive and transmit packets. When connected/connecting to PC, tuning command packets come in from AWE Server to the target via comm layer.

4. The incoming packet contents get written to the previously initialized packet buffer

5. In the main BSP idle loop, the BSP author checks if the packet has been completely received, and if so, calls awe\_pltTuningRxPacket to indicate to the AWECore that a packet is ready.

```cpp
BOOL bReplyReady;
if (g_bUSBPacketReceived)
{
 awe_pltTuningRxPacket();
}
```

6. Next, the BSP author calls awe\_pltTuningTick\(\). _Note: some packets are processed directly by this API, while others are routed to the awe\_fwPacketProcess function \(step 2\). The user does not need to know which API is processing the packets, but_ _**both**_ _need to be called for a successful tuning interface._

```cpp
bReplyReady = awe_pltTuningTick();
```

7. The return value of awe\_pltTuningTick\(\) informs the user if the latest packet has been processed completely by AWECore. When a packet is processed, the reply is now in the packet buffer, from which it is sent back to the PC via the comm layer.

```cpp
if (bReplyReady == REPLY_READY)
{
 USBSendReply();
}
```

**AWE Core 8**

Because AWE Core 8 removes the concept of “Framework vs Platform”, all tuning commands are processed by a single API.

1. The BSP author defines either a single UINT32 packet buffer of size 264 words, or two individual buffers for packet/reply. During initialization of the AWEInstance, these buffer variable ptrs are assigned to the AWEInstance.pPacketBuffer and AWEInstance.pReplyBuffer members. For a single packet buffer, assign the members to the same ptr as seen below.

```cpp
UINT32 g_packet_buffer[MAX_COMMAND_BUFFER_LEN] = {0};
…
g_AWEInstance.pPacketBuffer = g_packet_buffer;
g_AWEInstance.pReplyBuffer = g_packet_buffer; // Shared reply and receive buffers
```

2. The BSP author sets up some communication protocol layer code \(unrelated to AWE, usually USB, RS232, TCP,etc\) to send and receive packets.

3. When connected/connecting to the PC, packets come in from AWE Server via comm layer. The BSP author is responsible for writing the incoming packets to the memory assigned to AWEInstance.pPacketBuffer.

4. Once a complete packet has been received, the BSP author calls awe\_packetProcess\(\) and passes in the AWEInstance with its newly received tuning command.

```cpp
if (g_bPacketReceived)
{
 awe_packetProcess(&g_AWEInstance);
}
```

5. The processed reply packet is now contained in the AWEInstance.pReplyBuffer, and it is sent back to the PC via the comm protocol layer.

```cpp
USBSendReply(&g_AWEInstance);
```

\*\*\*\*

**Tuning API’s Summary**

#### Tuning Interface setup/initialization

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
void awe_fwPacketInit(UINT32 *packet_buf, UINT32 buf_length);

void awe_pltPacketInit(UINT32 * packet_buf, INT32 buf_length);

CoreDescriptor.m_coreID = AWEInstance.m_coreID
// BSP author must initialize CoreDescriptor struct and appropriately assign m_coreID to same value as AWEInstance.m_coreID

CoreDescriptor.m_pSendCommand = user defined callback
// BSP author must define a function to be used as a callback, and assign its function ptr to this member 
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
awe_init() 
// assumes buffer ptr’s are assigned to AWEInstance packet/reply buffer members, and instanceID member set appropriately (0 for single core).
```
{% endtab %}
{% endtabs %}

### Packet Processing

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
INT32 awe_pltTuningTick(void);
// Processes certain types of incoming tuning packets, and used to determine if tuning reply ready

INT32 awe_fwPacketProcess(AWEInstance * pAWE);
// Invoked in the required usr defined callback function set to CoreDescriptor. Processes whatever packets that awe_pltTuningTick() does not. 

void awe_pltTuningRxPacket(void);
// BSP author must call this in idle loop when complete packet is received from comm layer...must be called before awe_pltTuningTick().
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_packetProcess(AWEInstance * pAWE);
```
{% endtab %}
{% endtabs %}

### Audio Loop

AWE Core 6 and AWE Core 8 both share the concept of an audio pump, however there are some key improvements in AWE Core 8:

* Clearly named functions \(all audio API’s have “audio” in them\)
* Automatic channel matching and data type handling with new awe\_audioImportSamples and awe\_audioExportSamples functions.

Both versions use the following high level model.

1. Get input samples from the system’s audio driver to AWE Core buffers
2. Process the samples with “pump”
3. Write the processed samples from the AWE Core buffer back to the audio driver’s output buffer

**AWE Core 6**

1. Set up a DMA audio loop. Make sure that audio can be passed from the input directly to the output, independent of any AWE code.

2. In the audio loop, check if audio is started, and then get the channel count of the loaded AWE layout.

```cpp
if (!awe_pltIsAudioRunning() )
{
 // output zero’s is audio is not started
 memset(pProcessedSamples, 0, AWE_OUTPUT_BUFFER_SIZE_IN_SAMPLES *PCM_SAMPLE_SIZE_IN_BYTES);
}
else
{
 awe_fwGetChannelCount(&g_AWEInstance, nPinNdx, &fwInCount, &fwOutCount);
...
```

3. Compare the layout channels against the hardware input channels. This is required for manual channel matching.

4. Get the AWECore input channel buffer pointer with awe\_fwGetInputChannelPtr\(\). Loop through the available input channels. This is part of a process called “channel matching”, see the AWE6 docs and sample BSP’s for more info.

5. Convert incoming audio data to fract32 data type.

6. Once the data is formatted correctly, copy the contents of the audio DMA input buffer to the AWECore input buffer. If there are more channels in the layout than available on the hardware, insert zero’s for these channels.

```cpp
 awe_fwGetChannelCount(&g_AWEInstance, nPinNdx, &fwInCount, &fwOutCount);
 for (chan = 0; chan < fwInCount; chan++)
 {
    dest32BitPtr = awe_fwGetInputChannelPtr(&g_AWEInstance, nPinNdx, chan, &pinStride);
    if (chan < INPUT_CHANNEL_COUNT)
    {
       src16BitPtr = pMicSamples;
       for (nSample = 0; nSample < AWE_FRAME_SIZE; nSample++)
       {
          n32BitSample = *src16BitPtr; // Load 16-bit signed value into 32-bit register
          *dest32BitPtr = n32BitSample << 16; // Audio Weaver wants the sample in the high order bytes of the 32-bit word
          src16BitPtr += MONO_CHANNEL_COUNT;
          dest32BitPtr += pinStride;
       }
    }
 
    // Zero any unused layout inputs
    else
    {
       for (nSample = 0; nSample < AWE_FRAME_SIZE; nSample++)
       {
          *dest32BitPtr = 0;
          dest32BitPtr += pinStride;
       }
     }
 }
```

7. Repeat steps 3-5 for the output buffer.

```cpp
for (chan = 0; chan < OUTPUT_CHANNEL_COUNT; chan++)
{
    dest16BitPtr = pProcessedSamples + chan;
    if (chan < fwOutCount)
    {
        src32BitPtr = (fract32 *)awe_fwGetOutputChannelPtr(&g_AWEInstance, nPinNdx, chan, &pinStride);
        for (nSample = 0; nSample < AWE_FRAME_SIZE; nSample++)
        {
            n32BitSample = *src32BitPtr;
            n16BitSample = n32BitSample >> PCM_SAMPLE_SIZE_IN_BITS;
            *dest16BitPtr = n16BitSample;
            src32BitPtr += pinStride;
            dest16BitPtr += OUTPUT_CHANNEL_COUNT;
        }
    }
    else
    {
        for (nSample = 0; nSample < AWE_FRAME_SIZE; nSample++)
        {
            *dest16BitPtr = 0;
            dest16BitPtr += OUTPUT_CHANNEL_COUNT;
        }
    }
}
```

8. Let the AWECore know that the input buffers have been populated by calling awe\_fwAudioDMAComplete\(\).

```cpp
nComplete |= awe_fwAudioDMAComplete(&g_AWEInstance, nPinNdx, AWE_FRAME_SIZE);
```

9. Determine which sub-layouts are to be pumped with awe\_fwEvalLayoutMask\(\), which will return the layout mask.

10. Process the samples with the awe\_fwPump\(\) function for any sub-layout that is ready to be pumped \(determined from the layoutMask from the previous step\). These pump calls are usually made in lower priority interrupts, but are shown inline with the DMA interrupt below for simplicity.

```cpp
if (nComplete)
{
    layoutMask = awe_fwEvalLayoutMask(&g_AWEInstance, NULL);

    if (layoutMask & 1)
    {
        awe_fwPump(&g_AWEInstance, 0);
    }
    if (layoutMask & (1 << 1))
    {
        awe_fwPump(&g_AWEInstance, 1);
    }
}
```

**AWE Core 8**

AWE Core 8 does not require that the user implement any channel matching to support systems where the loaded layout’s input and output channel counts do not match the available hardware audio channels. This is handled automatically by the awe\_audioImportSamples and awe\_audioExportSamples functions.

1. Set up a DMA audio loop. Make sure that audio can be passed from the input directly to the output, independently of any AWE code.

2. In the DMA loop, check if audio is started with awe\_audioIsStarted\(\), and check if the loaded layout is valid with awe\_layoutIsValid\(\). If the audio isn’t started or the layout is invalid, handle appropriately \(up to BSP author, but either pass through or output 0’s directly to DMA\).

```cpp
bAudioIsStarted = awe_audioIsStarted(&g_AWEInstance);
bLayoutValid = awe_layoutIsValid(&g_AWEInstance);

if (!bAudioIsStarted)
{
    memset(I2SBuffer, 0, sizeof(I2SBuffer) );
}
else if (!bLayoutValid)
{
    memcpy(&I2SBuffer[nI2SBufferNdx], pUSBSamples, STEREO_BLOCK_SIZE_IN_SAMPLES * PCM_SIZE_IN_BYTES);
}
else
{
...
```

3. Call awe\_audioImportSamples\(\) for each hardware input channel, which writes the audio data from each DMA input buffer to the corresponding AWECore input channel buffer. This API takes arguments to determine which channel, type, and stride \(for interleaving purposes\).

```cpp
awe_audioImportSamples(&g_AWEInstance, pUSBSamples, STRIDE2, CHANNEL1, Sample16bit);
awe_audioImportSamples(&g_AWEInstance, &pUSBSamples[1], STRIDE2, CHANNEL2, Sample16bit);
awe_audioImportSamples(&g_AWEInstance, &PCMBuffer[nPCMBufferNdx], STRIDE1, CHANNEL3, Sample16bit);
```

4. Determine which sublayout is ready to be pumped with awe\_audioGetPumpMask\(\). This function returns the pump layoutMask, similar to AWE6.

```cpp
layoutMask = awe_audioGetPumpMask(&g_AWEInstance);
```

5. Using layoutMask, process the samples for the appropriate layouts with awe\_audioPump\(\). This function will return a 1 if deferred processing is required\(see Deferred Processing section\)

```cpp
if (layoutMask > 0)
{
    if (layoutMask & 1)
    {
        g_bDeferredProcessingRequired |= awe_audioPump(&g_AWEInstance, 0);
    }
    if (layoutMask & (1 << 1))
    {
        g_bDeferredProcessingRequired |= awe_audioPump(&g_AWEInstance, 1);
    
    }
}
```

6. Call awe\_audioExportSamples\(\) for each hardware output channel to write the processed audio from AWECore back to the DMA buffer.

```cpp
awe_audioExportSamples(&g_AWEInstance, &I2SBuffer[nI2SBufferNdx], STRIDE2, CHANNEL1, Sample16bit);
awe_audioExportSamples(&g_AWEInstance, &I2SBuffer[nI2SBufferNdx + 1], STRIDE2, CHANNEL2, Sample16bit);
```

**Audio API’s Summary**

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
INT32 awe_fwPump(AWEInstance *pAWE, UINT32 layout_no);

INT32 *awe_fwGetInputChannelPtr(const AWEInstance *pAWE, UINT32 pinIdx, UINT32 chan, INT32 *stride);

INT32 *awe_fwGetOutputChannelPtr(const AWEInstance *pAWE, UINT32 pinIdx, UINT32 chan, INT32 *stride);

UINT32 awe_fwEvalLayoutMask(const AWEInstance *pAWE, UINT32 dividers[4]);

BOOL awe_pltIsAudioRunning(void);

UINT32 awe_fwAudioDMAComplete(const AWEInstance *pAWE, UINT32 pinIdx, INT32 samplesPerTick);
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_audioPump(AWEInstance *pAWE, UINT32 layoutIndex);

INT32 awe_audioImportSamples(const AWEInstance *pAWE, const void *inSamples, INT32 inStride, INT32 channel, SampleType inType);

INT32 awe_audioExportSamples(const AWEInstance *pAWE,  void *outSamples, INT32 outStride, INT32 channel, SampleType outType);

INT32 awe_audioGetPumpMask(const AWEInstance *pAWE);

INT32 awe_audioIsStarted(const AWEInstance *pAWE);

N/A - awe_audioGetPumpMask handles this
```
{% endtab %}
{% endtabs %}

#### 

### Control Interface

In both AWE6 and AWE8, the control interface API’s are used to access module variables directly from the application code. A simple use case of the control interface is to set the gain variable of a scaler module in the layout via a GPIO volume knob.

Both 6 and 8 allow the user to access module variables using a module handle, generated by the Tools -&gt; Generate Target Files menu in Designer. Select the ‘ControlInterface.h’ option when generating target files for the layout, and any module that has been assigned an object ID will have module and variable handles defined in the generated file. Module handle addresses differ slightly between AWE 6 and AWE 8, so 2 files are generated when using this tool. As might be expected, the ‘ControlInterface\_AWE6.h’ file should be used for AWE6 products, and the ‘ControlInterface.h’ file should be used for anything else.

**AWE Core 6**

AWE 6 has separate functions for setting and getting scalar variables and array variables. Additionally, the final module handles must be calculated by the user using the macro MAKE\_ADDRESS defined in Framework.h.

The scalar access functions, awe\_fwSetValueSetCall\(\) and awe\_fwGetCallFetchValue\(\), require that the Sample union type, defined in Framework.h, be used to set and get the module variable. The array access functions, awe\_fwSetValuesSetCall and awe\_fwGetCallFetchValues, can use pointers and casting.

In any example code below, the ‘AWE\_Scaler’ macros are defined in the ‘ControlInterface\_AWE6.h’ file generated in Designer.

```cpp
  Sample tmp;
  INT32 ret;
  
  // Set AWE_Scaler1_gain.
  // AWE_Scaler1_gain is type float, use Sample union.
  tmp.fVal = -10.0f;
  awe_fwSetValueSetCall(&g_AWEInstance, MAKE_ADDRESS(AWE_Scaler1_ID, AWE_Scaler1_gain_OFFSET), tmp.iVal, AWE_Scaler1_gain_MASK, 0, 1);
  
  // Check that gain was set correctly
  // AWE_Scaler1_gain is type float, use Sample union.
  tmp.iVal = awe_fwGetCallFetchValue(&g_AWEInstance, MAKE_ADDRESS(AWE_Scaler1_ID, AWE_Scaler1_gain_OFFSET), AWE_Scaler1_gain_MASK, &ret, 0, 1);
  assert(tmp.fVal == -10.0f);

  // Get AWE_Meter1_value
  // AWE_Meter1_value[] is type float. Cast to Sample * to make the compiler happy
  awe_fwGetCallFetchValues(&AWE, MAKE_ADDRESS(AWE_Meter1_ID, AWE_Meter1_value_OFFSET), 0, AWE_Meter1_value_MASK, AWE_Meter1_value_SIZE, (Sample *)AWE_Meter1_value, 1);
```

 

Attempting to access a module variable that is not present in the loaded layout can lead to undefined behavior, so it is often wise to check that the module exists before any control interface functions are used. AWE Core 6 can check for the existence of a module as below:

```cpp
 INT32 ret;
 InstanceDescriptor *ptr;
 UINT32 classID;
 
 // Check that AWE_Scaler1_ID exists
 ret = awe_fwGetObjectByID(&g_AWEInstance, AWE_Scaler1_ID, &ptr, &classID);
 if (ret < 0)
 {
     printf("Object AWE_Scaler1_ID=%d not found - failed with %d\n", AWE_Scaler1_ID, ret);
 }
 else if (classID != CLASS_OF_ScalerV2)
 {
     printf("Object AWE_Scaler1_ID is not an instance of ScalerV2\n");
 }
```

**AWE Core 8**

The control interface functions in AWE 8 all begin with ‘awe\_ctrl’. Rather than having different functions for handling arrays and scalar module variables like in AWE 6, the AWE 8 control interface access functions all take a size argument. In the case of scalar variables, this size argument is always one.

In any example code below, the ‘AWE\_Scaler’ macros are defined in the ‘ControlInterface.h’ file generated in Designer.

An example of setting and then retrieving the value of the gain variable in a Scaler module is shown here:

```cpp
 // Set scaler gain to -10 dB
 scaler1Gain = -10.0;
 awe_ctrlSetValue(&aweInstance, AWE_Scaler1_gain_HANDLE, (const void *)&scaler1Gain, 0, AWE_Scaler1_gain_SIZE);
 
 // Make sure it got set correctly
 awe_ctrlGetValue(&aweInstance, AWE_Scaler1_gain_HANDLE, &scaler1Gain, 0, AWE_Scaler1_gain_SIZE);
 assert(scaler1Gain == -10.0);
```



Before accessing a module’s variables, it is usually wise to first check to make sure that the expected module exists in the loaded layout. This is done in AWE Core 8 using the ‘awe\_ctrlGetModuleClass’ function, as shown below for the Scaler module accessed above:

```cpp
ret = awe_ctrlGetModuleClass(&aweInstance, AWE_Scaler1_gain_HANDLE, &classId) == E_SUCCESS)
if (E_SUCCESS == ret)
{
    if (AWE_Scaler1_classID != classId )
    {
        // Error: Module classID’s do not match
    }
}
else
{
    // Error: Module DNE in loaded layout, do not access
}
```

For advanced users, AWE Core 8 also provides functions that can control a module’s status \(bypass, mute, etc\), and functions that provide more control while getting and setting module variables by using variable masks. See the AWE Core 8 API documentation for more details.

**Control Interface API’s Summary**

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
INT32 awe_fwGetObjectByID(const AWEInstance *pAWE, UINT32 ID, InstanceDescriptor **pObject, UINT32 *pClassID);

INT32 awe_fwSetValuesSetCall(const AWEInstance *pAWE, UINT32 address, UINT32 offset, UINT32 mask, UINT32 argCount, const Sample *args, UINT32 floatFlg);

INT32 awe_fwSetValueSetCall(const AWEInstance *pAWE, UINT32 address, INT32 value, UINT32 mask, UINT32 ptrOffset, UINT32 floatFlg);

INT32 awe_fwGetCallFetchValue(const AWEInstance *pAWE, UINT32 address, UINT32 mask, INT32 *retVal, UINT32 ptrOffset, UINT32 floatFlg);

INT32 awe_fwGetCallFetchValues(const AWEInstance *pAWE, UINT32 address, UINT32 offset, UINT32 mask, UINT32 argSize, Sample *args, UINT32 floatFlg);

#define MAKE_ADDRESS(objectID, index)        ((objectID<<12) | index)
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_ctrlGetModuleClass(const AWEInstance *pAWE, UINT32 handle, UINT32 *pClassID);

INT32 awe_ctrlSetValue(const AWEInstance *pAWE, UINT32 handle, const void *value, INT32 arrayOffset, UINT32 length);

INT32 awe_ctrlGetValue(const AWEInstance *pAWE, UINT32 handle, void *value, INT32 arrayOffset, UINT32 length);

N/A - not required
```
{% endtab %}
{% endtabs %}

#### 

### Flash File System

Both AWE Core 6 and AWE Core 8 have a Flash File System feature, but with some key differences in function/implementation. As flash memory access varies greatly on different platforms, both AWE6 and AWE8 expect the user to define their own logic for reading/writing flash memory on the platform.

**AWE Core 6**

In AWE 6, the user is required to define some “user functions”. These function signatures must exactly match the function signatures in the Filesystem.h header. Even if the BSP does not utilize an FFS, these functions must be defined and stubbed out to satisfy the linker. For convenience, the provided ‘aweOptionalFeatures.h’ header stubs these functions out and can be included in systems not defining a Flash File System. AWE Core 6 will conditionally include this file to satisfy the linker if the BSP author initializes their AWEInstance’s targetinfo “isFlash” arg to false. When set to true, the user is expected to define their own user functions, or else the linker will fail.

1. First, tell the AWEInstance that an FFS will be implemented by passing in TRUE for the “isFlash” argument to awe\_fwInitTargetInfo \(‘HAS\_FLASH\_FILESYSTEM’ below\).

```cpp
#define HAS_FLASH_FILESYSTEM 1
awe_fwInitTargetInfo(&g_AWEInstance,
                        CORE_ID,
                        CORE_SPEED,
                        SAMPLE_SPEED,
                        "ST32F407",
                        PROCESSOR_TYPE_CORTEXM4,
                        HAS_FLOAT_SUPPORT,
                        HAS_FLASH_FILESYSTEM,
                        NO_HW_INPUT_PINS,
                        NO_HW_OUTPUT_PINS,
                        IS_SMP,
                        NO_THREADS_SUPPORTED,
                        FIXED_SAMPLE_RATE,
                        INPUT_CHANNEL_COUNT,
                        OUTPUT_CHANNEL_COUNT,
                        VER_DAY, VER_MONTH, VER_YEAR,
                        AWE_FRAME_SIZE,
                        MAX_COMMAND_BUFFER_LEN
                        );
```

2. Define the required awe\_pltInitFlashFileSystem, and initialize any platform specific flash features as needed. At the end of this function, call the AWECore API InitFlashFileSystem\(\) which will actually initialize the FFS portion of the AWE instance.

```cpp
AWE_OPTIMIZE_FOR_SPACE
AWE_FW_SLOW_CODE
BOOL awe_pltInitFlashFileSystem(void)
{
    // Clear the flash register flags
    __HAL_FLASH_CLEAR_FLAG( (FLASH_FLAG_EOP | \
                             FLASH_FLAG_OPERR | FLASH_FLAG_WRPERR | FLASH_FLAG_PGAERR | \
                             FLASH_FLAG_PGPERR | FLASH_FLAG_PGSERR | FLASH_FLAG_RDERR));

      g_filesystem_info.m_FlashDeviceDWords = FLASH_MEMORY_SIZE_IN_WORDS;

    return InitFlashFileSystem(FLASH_MEMORY_SIZE_IN_BYTES,
                               ERASEABLE_BLOCKSIZE,
                               FILE_SYSTEM_START_OFFSET);
}
```

3. Define the rest of the required user functions and populate with the relevant flash code for the platform.

```cpp
BOOL awe_pltReadFlashMemory(UINT32 nFlashAddress, UINT32 * pBuffer, UINT32 nDWordsToRead)
{
    ...
    
        for (n = 0; n < nDWordsToRead; n++)
        {
            pBuffer[n] = pSrc[n];
        }

        return SUCCESS;

}

BOOL awe_pltWriteFlashMemory(UINT32 nFlashAddress, UINT32 * pBuffer, UINT32 nDWordsToWrite)
{
    ...

        if (nDWord != pBuffer[n])
        {
            return FAILURE;
        }
        nFlashAddress = nFlashAddress + 4;
        n++;
    ...
        else
        {
            return FAILURE;
        }
    }
        return SUCCESS;
        
}

BOOL awe_pltEraseFlashMemorySector(UINT32 nStartingAddress, UINT32 nNumberOfSectors)
{
    ...

    if (HAL_FLASHEx_Erase(&EraseInitStruct, (uint32_t *)&SectorError) != HAL_OK)
    {
        return FAILURE;
    }
    ...

        return SUCCESS;
}

INT32 awe_pltGetFlashEraseTime(void)
{
    return 0;
}
```

**AWE Core 8**

In AWE Core 8, the flash file system has its own instance structure, AWEFlashFSInstance. Unlike AWE 6 which relies on precisely matching function signatures, this structure is initialized with ptrs to the user defined functions.

1. Define an AWEFlashFSInstance

```cpp
AWEFlashFSInstance g_AWEFlashFSInstance;
```

2. Memset the AWEFlashFSInstance to 0’s, then set the required member variables.

```cpp
memset(&g_AWEFlashFSInstance, 0, sizeof(AWEFlashFSInstance) );
g_AWEFlashFSInstance.flashSizeInBytes = FLASH_MEMORY_SIZE_IN_BYTES;
g_AWEFlashFSInstance.flashErasableBlockSizeInBytes = ERASEABLE_BLOCKSIZE;
g_AWEFlashFSInstance.flashStartOffsetInBytes = FILE_SYSTEM_START_OFFSET;
g_AWEFlashFSInstance.flashEraseTimeInMs = 0;
```

3. Write the required read/write/erase/init flash user functions, and assign them to the corresponding AWEFlashFSInstance members.

```cpp
g_AWEFlashFSInstance.cbInit = &usrInitFlashFileSystem;
g_AWEFlashFSInstance.cbEraseSector = &usrEraseFlashMemorySector;
g_AWEFlashFSInstance.cbFlashWrite = &usrWriteFlashMemory;
g_AWEFlashFSInstance.cbFlashRead = &usrReadFlashMemory;

BOOL usrInitFlashFileSystem(void)
{
    __HAL_FLASH_CLEAR_FLAG( (FLASH_FLAG_EOP | \
                             FLASH_FLAG_OPERR | FLASH_FLAG_WRPERR | FLASH_FLAG_PGAERR | \
                             FLASH_FLAG_PGPERR | FLASH_FLAG_PGSERR));
    
    return SUCCESS;
}

BOOL usrReadFlashMemory(UINT32 nFlashAddress, UINT32 * pBuffer, UINT32 nDWordsToRead)
{
    …

    for (n = 0; n < nDWordsToRead; n++)
    {
            pBuffer[n] = pSrc[n];
    }

    return SUCCESS;
}

BOOL usrWriteFlashMemory(UINT32 nFlashAddress, UINT32 * pBuffer, UINT32 nDWordsToWrite)
{
    ...

    if (HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, nFlashAddress, pBuffer[n]) == HAL_OK)
    {
        // Read back DWORD that was written
        DWORD nDWord;
        nDWord = *(PDWORD)nFlashAddress;
        if (nDWord != pBuffer[n])
        {
            return FAILURE;
        }

    ...

    return SUCCESS;
}

BOOL usrEraseFlashMemorySector(UINT32 nStartingAddress, UINT32 nNumberOfSectors)
{
    ...

    if (nStartingAddress < ADDR_FLASH_SECTOR_7)
    {
        nSector_to_erase = FLASH_SECTOR_6;
    }

    ...

    __HAL_FLASH_DATA_CACHE_DISABLE();
    __HAL_FLASH_DATA_CACHE_RESET();
    __HAL_FLASH_DATA_CACHE_ENABLE();

        return SUCCESS;
}
```

4. MUTI-CORE/MULTI-INSTANCE ONLY: Write the packet processing callback and assign it to the AWEFlashFSInstance.cbFlashProcessCmd. This processing callback will handle tuning commands from flash in the same way that they are handled in a multi-instance tuning interface \(parse instanceID, call packetProcess on the desired AWEInstance\).

```cpp
INT32 usrProcessPacket(AWEInstance * pAWE)
{
    INT32 opcode;
    UINT32 packetCoreID;
    INT32 nStatus = E_SUCCESS;

    opcode = PACKET_OPCODE(pAWE->pPacketBuffer);
    packetCoreID = PACKET_INSTANCEID(pAWE->pPacketBuffer);

    if (opcode == PFID_GetCores2)
    {
        GenerateInstanceTableReply(pAWE->pPacketBuffer, NUM_INSTANCES, g_InstanceIDs);
    }
    else if (packetCoreID == 0)
    {
        awe_packetProcess(pAWE);
    }
    else if (packetCoreID == 16)
    {
        nStatus = Transact(pAWE->pPacketBuffer, pAWE->pReplyBuffer);
    }

    return nStatus;

}	
```

5. After calling awe\_init\(\), Call awe\_initFlashFS\(\) to initialize the AWEFlashFSInstance.

```cpp
awe_initFlashFS(&g_AWEInstance, &g_AWEFlashFSInstance);
```

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
BOOL InitFlashFileSystem(UINT32 nFlashMemorySizeBytes, UINT32 nEraseableBlockSize, UINT32 nFileSystemStartOffset);
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
void awe_initFlashFS(AWEInstance * pAWE, AWEFlashFSInstance * pAWEFlashFSInstance);
```
{% endtab %}
{% endtabs %}

#### 

### Deferred Processing

Some AWE modules require additional calculations that do not need to be run as part of the audio processing, and need to defer some of these calculations from the main audio process to a lower priority process. For example, many SOF filter modules will recalculate the biquad coefficients in these background threads as cutoff frequency changs. AWE Core 6 and 8 both handle this in an automatic and intelligent feature called “Deferred Processing”.

**AWE Core 6**

In AWE Core 6, a BSP author is required to continuously call the API function awe\_fwTick\(\) in their low priority idle loop \(or some other low priority process\).

```cpp
while(TRUE)
{
    awe_fwTick(&g_AWEInstance);
}
```

There is no way for the author to know if there is any actual deferred processing to be done, as this varies from layout to layout. Calling awe\_fwTick\(\) constantly \(even when there is no deferred processing pending\) can cause unnecessary loading of the CPU.

**AWE Core 8**

There are a few key differences regarding Deferred Processing between AWE Core 6 vs 8. Firstly, the awe\_fwTick\(\) function is renamed to the more descriptive, awe\_deferredSetCall \(‘SetCall’ because it calls the module’s Set function\).

Another key difference is that deferred processing can be invoked conditionally based on the need for any pending work. The BSP author is able to determine if there is any pending work required with the return value of awe\_audioPump\(\). The awe\_audioPump\(\) API will return 1 if there is any work to be done, and 0 if not.

This gives the BSP author flexibility to conditionally invoke deferred processing by calling awe\_deferredSetCall\(\) from wherever they please. For example...

* Signal to another low priority thread from the audio loop
* Directly in the audio loop
* Constantly in the idle loop

awe\_deferredSetCall\(\) only calls the Set function of a single module each time it is called. This means that it may need to be called multiple times after a single pump if multiple modules require deferred processing. To handle this, it returns 1 if there are still more Set functions that need to be done, and 0 when it is finished. The BSP author can simply call this once per cycle, or use a while loop to invoke the API as many times as it needs to be invoked.

Signal to low priority thread and handle all deferred work until it is done, via a while loop.

```cpp
//audio loop
g_bDeferredProcessingRequired = awe_audioPump(&g_AWEInstance, 0);
...
//low priority thread
if (g_bDeferredProcessingRequired == 1)
{
    while(bMoreProcessingRequired == 1)
    {
        bMoreProcessingRequired = awe_deferredSetCall(&g_AWEInstance);
    }
}
```

 Call directly in the audio loop, one tick at a time.

```cpp
g_bDeferredProcessingRequired = awe_audioPump(&g_AWEInstance, 0);
if (g_bDeferredProcessingRequired == 1)
{
    awe_deferredSetCall(&g_AWEInstance);
}
```

Continuously in idle loop "AWE6 style".

```cpp
//idle loop
if (g_bDeferredProcessingRequired || bMoreProcessingRequired)
{
    g_bDeferredProcessingRequired = FALSE;
    bMoreProcessingRequired = awe_deferredSetCall(&g_AWEInstance);
}
```

**Deferred Processing Summary**

{% tabs %}
{% tab title="AWE Core 6" %}
```cpp
INT32 awe_fwTick(AWEInstance *pAWE);
```
{% endtab %}

{% tab title="AWE Core 8" %}
```cpp
INT32 awe_deferredSetCall(AWEInstance * pAWE);
```
{% endtab %}
{% endtabs %}

### 

## AWELib -&gt; AWE Core OS

### Initialization

AWELib has a two step initialization process. The first step creates a CAWELib class instance with `AWELibraryFactor()`, and the second step initializes the AWELib instance with the passed in parameters. Since it has a built in, immutable list of modules, the AWELib initialization does not require any information about the module list.

AWE Core OS also has a two step initialization process, the first step being to configure the instance. A `AWEOSConfigParameters` structure can be initialized with `aweOS_getParamDefaults` __and then updated as needed for that particular instance. In addition to this configuration structure,   `aweOS_init` requires an opaque `AWEOSInstance` type that hides the internal instance variables, and some information about the list of modules that will be linked-in to the application.

**AWELib**

```cpp
#include "awelib.h"

// Create audio start/stop callback functions
static void OnAudioStop(void * /* param */)
{
        …
}

static void OnAudioStart(void * /* param */)
{
        …
}

// Create and initialize AWELib instance
CAWELib *pAwelib = AWELibraryFactory();
pAwelib->CreateEx("libtest", cpu_freq, prof_freq, profile, blocksize, bufferlen, inchans, outchans);

pAwelib->SetStartStopCallbacks(NULL, OnAudioStart, OnAudioStop);
```

**AWE Core OS**

```cpp
#include “AWE Core OS.h”
#include “ModuleList.h”

...
// Declare global AWEOS variables
AWEOSInstance *g_AWEOSInstance;
static AWEOSConfigParameters configParams;

// Declare a module descriptor table. This module descriptor table includes the modules as defined in ModuleList.h
static const void* moduleDescriptorTable[] =
{
    LISTOFCLASSOBJECTS
};
UINT32 moduleDescriptorTableSize = sizeof(moduleDescriptorTable) / sizeof(moduleDescriptorTable[0]);

// Create audio start/stop callback functions
INT32 aweOSuser_audioStop(AWEOSInstance *pAWEOS)
{
        ...
}

INT32 aweOSuser_audioStart(AWEOSInstance *pAWEOS)
{
        ...
}
…

// Get default configuration parameters then update config as desired. See AWE Core OS.h for default values
aweOS_getParamDefaults(&configParams);

// Set audio start/stop function pointers
configParams.cbAudioStart = aweOSuser_audioStart;
configParams.cbAudioStop = aweOSuser_audioStop;

// Initialize the AWEOSInstance with config structure and module table parameters
// Note that the AWEOSInstance argument is a double pointer
aweOS_init(&g_AWEOSInstance, &configParams, moduleDescriptorTable, moduleDescriptorTableSize);
```

#### 

### Loading Layouts

Loading layouts is done in largely the same way between AWELib and AWE Core OS. Both products support loading AWB’s stored on the filesystem or compiled directly into the application as a C array. These .awb or .c files are generated in the same way for both products using the ‘Tools-&gt;Generate Target Files’ menu in Designer.

In addition to the functions shown below, AWELib has multiple other functions for loading AWB’s that differ depending on whether or not the AWB is encrypted. AWE Core OS has only a single set of functions that handle encrypted and unencrypted AWB’s.

#### **AWELib**

To load an AWB from an .awb file on the filesystem, use `LoadAwbFile`:

```cpp
UINT32 pos;
pAwelib->LoadAwbFile(awbFullFilePath, &pos);
```

To load an AWB compiled as a C array, use `LoadAwbMem`

```cpp
// Include header file for corresponding AWB.c file that has been compiled as part of the same executable. This defines Core0_InitCommands used below
#include "passthrough_InitAWB.h"
…
UINT32 pos;
pAwelib->LoadAwbMem(Core0_InitCommands, Core0_InitCommands_len, &pos);
```

After loading an AWB, the layout information can be queried using `PinProps` and then the wireIds can be propagated to the instance with `SetLayoutAddresses`.

```cpp
UINT32 wireId1 = 1;
UINT32 wireId2 = 2;
UINT32 layoutId = 0;
UINT32 inCount, outCount, in_samps, in_chans, out_samps, out_chans, in_complex, out_complex;
pAwelib->PinProps(in_samps, in_chans, in_complex, wireId1,
                       out_samps, out_chans, out_complex, wireId2);
pAwelib->SetLayoutAddresses(wireId1, wireId2, layoutId);
```

**AWE Core OS**

Loading a AWB from an .awb file on the filesystem can be done as follows:

```cpp
UINT32 position;
aweOS_loadAWBFile(g_AWEOSInstance, awbFullFilePath, &position);
```



Loading an AWB that has been compiled into a C array could look like this:

```cpp
// Include header file for corresponding AWB.c file that has been compiled as part of the same executable. This defines Core0_InitCommands used below
#include "passthrough_InitAWB.h"

…

UINT32 position;
aweOS_loadAWBFromArray(g_AWEOSInstance, Core0_InitCommands, Core0_InitCommands_len, &pos);

Once an AWB has been loaded, information about the layout can be retrieved using the functions that start with ‘aweOS_layout*’.

UINT32 layoutInChannels, layoutOutChannels, layoutBlockSize;
FLOAT32 layoutSampleRate;
aweOS_layoutGetChannelCount(g_AWEOSInstance, &layoutInChannels, &layoutOutChannels);
aweOS_layoutGetBlockSize(g_AWEOSInstance, &layoutBlockSize);
aweOS_layoutGetSampleRate(g_AWEOSInstance, &layoutSampleRate);
```

### Tuning Interface

AWELib requires that the user write their own tuning interface. For the majority of the targets, a TCP/IP style tuning interface is used, and the same provided reference code is simply copied to each application. Instead of requiring that the user create a tuning interface as part of their application, AWE Core OS includes an integrated TCP/IP tuning interface that handles all packet transmission and processing behind the scenes.

**AWELib**

In addition to the AWELib library, any application using the provided `NotifyFunction` TCP/IP tuning interface must also link the Utils library to access the TcpIO class used below. They must also copy the `NotifyFunction` source into the application.

```cpp
#include "TcpIO2.h"

static CTcpIO2 *ptcpIO = 0;
static UINT32 commandBuf[MAX_COMMAND_BUFFER_LEN];
static UINT32 s_partial = 0;
static UINT32 s_pktLen = 0;

…
// Copy reference implementation of TCP/IP tuning interface from libtest.cpp
static void NotifyFunction(void *pAwe, int count)
{
        CAWELib *pAwelib = (CAWELib *)pAwe;
        SReceiveMessage *pCurrentReplyMsg = ptcpIO->BinaryGetMessage();
        const int msglen = pCurrentReplyMsg->size();

        if (msglen != 0)
        {
                // This is a binary message.
                UINT32 *txPacket_Host = (UINT32 *)(pCurrentReplyMsg->GetData());
                if (txPacket_Host)
                {
                        UINT32 first = txPacket_Host[0];
                        UINT32 len = (first >> 16);
                        if (s_pktLen)
                        {
                                len = s_pktLen;
                        }
                        if (len > MAX_COMMAND_BUFFER_LEN)
                        {
                                printf("count=%d msglen=%d\n",
                                        count, msglen);
                                printf("NotifyFunction: packet 0x%08x len %d too big (max %d)\n", first,
                                        len, MAX_COMMAND_BUFFER_LEN);
                                exit(1);
                        }
                        if (len * sizeof(int) > s_partial + msglen)
                        {
                                // Paxket is not complete, copy partial words.
//                              printf("Partial message bytes=%d len=%d s_partial=%d\n",
//                                      msglen, len, s_partial);
                                memcpy(((char *)commandBuf) + s_partial, txPacket_Host, msglen);
                                s_partial += msglen;
                                s_pktLen = len;
                                return;
                        }
                        memcpy(((char *)commandBuf) + s_partial, txPacket_Host, msglen);
                        s_partial = 0;
                        s_pktLen = 0;

                        // AWE processes the message.
                        int ret = pAwelib->SendCommand(commandBuf, MAX_COMMAND_BUFFER_LEN);
                        if (ret < 0)
                        {
                                printf("NotifyFunction: SendCommand failed with %d\n", ret);
                                exit(1);
                        }

                        // Verify sane AWE reply.
                        len = commandBuf[0] >> 16;
                        if (len > MAX_COMMAND_BUFFER_LEN)
                        {
                                printf("NotifyFunction: reply packet len %d too big (max %d)\n",
                                len, MAX_COMMAND_BUFFER_LEN);
                                exit(1);
                        }
                        ret = ptcpIO->SendBinaryMessage("", -1, (BYTE *)commandBuf, len * sizeof(UINT32));
                        if (ret < 0)
                        {
                                printf("NotifyFunction: SendBinaryMessage failed with %d\n", ret);
                                exit(1);
                        }
                }
                else
                {
                        printf("NotifyFunction: impossible no message pointer\n");
                        exit(1);
                }
        }
        else
        {
                printf("NotifyFunction: illegal zero klength message\n");
                exit(1);
        }
}

...


// Start the TCP/IP thread on port 15002
UINT32 listenPort = 15002;
ptcpIO = new CTcpIO2();
ptcpIO->SetNotifyFunction(pAwelib, NotifyFunction);
HRESULT hr = ptcpIO->CreateBinaryListenerSocket(listenPort);
```

**AWE Core OS**

After `aweOS_init` has been called, open the built in tuning interface. Note that a double pointer is required for the AWEOSInstance argument.

```cpp
UINT32 tuningPort = 15002;
aweOS_tuningSocketOpen(&g_AWEOSInstance, tuningPort, 1);
```

To stop receiving tuning messages, the tuning interface can also be closed without destroying the AWEOSInstance.

```cpp
aweOS_tuningSocketClose(g_AWEOSInstance);
```

### Audio Loop

AWELib has a single PumpAudio\(\) function that expects users to manually format their audio data \(correct type, correct sample count \[blocksize\*channels\]\), and perform channel matching \(zeroing/nulling unused channels\) whenever needed.

AWE Core OS introduces new importSamples/exportSamples API’s that are called once per channel. These API’s take arguments to specify which one of the [supported audio data types](https://dspconcepts.s3-us-west-1.amazonaws.com/awecoreos/Docs/AWECoreOS_UserGuide/index.html#rtaudio_sample_formats) is being passed in, and then all of the required conversion happens under the hood. The AWE Core OS API is designed to match the approach of the AWECore 8 audio functions, so both AWE 8 products will have similar audio loops.

**Note about multi-threading/clock dividers:** Both products will pump all the sub-layouts included in the loaded layout automatically, so none of the multi-thread handling code needed in AWECore is required here. AWELib will pump these sub-layouts sequentially, while AWE Core OS will process these sub-layouts in parallel \(when possible\) in background threads.

**AWELib**

The AWELib audio processing loop is driven using a single function, ‘PumpAudio’. This function takes a pointer to the input buffer and output buffers, as well as the size of each buffer, and processes the loaded layout. These input and output buffers and the audio data they contain must be formatted exactly correctly before calling pump.

The layout block size must match the block size of the DMA, which we refer to as “fundamental” block size. For example, a layout of block size 512 must be run on an AWELib with a fundamental of 512. The samples must also accurately reflect channel count. “Channel matching” must also be implemented, which requires nulling out unused channels for a system where the layout’s channel count != the callback’s channel count.

```cpp
int audio_callback(const void *paInputBuffer, void *paOutputBuffer,
                         unsigned long framesPerBuffer,
                         const PaStreamCallbackTimeInfo* timeInfo,
                         PaStreamCallbackFlags statusFlags,
                         void *userData )
{
    /*suppress warnings for unused variables timeinfo, userData */
    (void)timeInfo;
    (void)userData;

    UINT32 chan = 0;
    UINT32 nSample = 0;

    /*
    Copy the data from the portaudio input buffer to the Audio Weaver input buffer. When channel
    counts mismatch, either ignore excess PA channels or zero excess AWE Audio Weaver channels
    */
    for (chan = 0; chan < channelStruct.used_inChans; chan++)
    {
        for (nSample = 0; nSample < framesPerBuffer; nSample++)
        {
            aweInputBuffer[nSample * channelStruct.layoutInChans + chan] = ((INT32*)paInputBuffer)[nSample * channelStruct.HW_inChans + chan];
        }
    }
    /*
    Zero out the "unused" input channels.
    */
    for ( ; chan < channelStruct.layoutInChans; chan++)
    {
        for (nSample = 0; nSample < framesPerBuffer; nSample++)
        {
            aweInputBuffer[nSample * channelStruct.layoutInChans + chan] = 0;
        }
    }
    
    /*
    Pump the audioweaver input buffer through AWELib then write the processed data to the audioweaver output buffer.
    */
    int callbackError = 0;
    callbackError = pAwelib->PumpAudio(aweInputBuffer, aweOutputBuffer, channelStruct.layoutInChans * framesPerBuffer, channelStruct.layoutOutChans * framesPerBuffer);
    if (callbackError > 0)
    {
        cout << "Error pumping audio. Error code # is: " << callbackError << " Exiting program..." << endl;
        exit(1);
    }
    
    /*
    Copy the data from the Audio Weaver output buffer to the PA output buffer. When channel
    counts mismatch, either zero excess PA channels or ignore excess AWE Audio Weaver channels
    */
    for (chan = 0; chan < channelStruct.used_outChans; chan++)
    {
        for (nSample = 0; nSample < framesPerBuffer; nSample++)
        {
            ((INT32*)paOutputBuffer)[nSample * channelStruct.HW_outChans + chan] = aweOutputBuffer[nSample * channelStruct.layoutOutChans + chan];
        }
    }
    /*
    Zero out the "unused" input channels.
    */
    for ( ; chan < channelStruct.HW_outChans; chan++)
    {
        for (nSample = 0; nSample < framesPerBuffer; nSample++)
        {
            ((INT32*)paOutputBuffer)[nSample * channelStruct.HW_outChans + chan] = 0;
        }
    }

    /*check for portaudio statusFlag errors (underflow, overflow, etc)*/
    if (statusFlags != 0)
    {
        cout << "Status flag is: " << statusFlags << endl;
    }

    return paContinue;
}
```

\*\*\*\*

**AWE Core OS**

AWE Core OS has a “double buffer” model and supports layouts with larger block sizes than the fundamental. For example, a layout with blocksize 512 and fundamental of 256 will run two DMA audio callbacks before actually pumping internally. Instead of manually doing blocksize \* channel count like AWELib, AWE Core OS provides the awe\_audioImportSamples and awe\_audioExportSamples API’s, which are called once per channel in the audio loop.

In order to simplify multiple audio devices, deinterleaved audio streams, or non Fract32 \(Q1.31\) audio, AWE Core OS exposes import and export audio functions that handle the input and output of the audio to/from the AWEOSInstance. These functions always operate at the ‘fundamentalBlockSize’ used in the configuration structure passed into the ‘aweOS\_init’ function. The user specifies which of the [supported audio data types](https://dspconcepts.s3-us-west-1.amazonaws.com/awecoreos/Docs/AWECoreOS_UserGuide/index.html#rtaudio_sample_formats) they are passing in, and then samples are internally converted to Fract32. See the code snippet below:

```cpp
//portaudio audio callback function. This is where the importing/exporting/pumping will happen
int audio_callback(const void *paInputBuffer, void *paOutputBuffer,
    unsigned long framesPerBuffer,
    const PaStreamCallbackTimeInfo* timeInfo,
    PaStreamCallbackFlags statusFlags,
    void *userData)
{
    /*suppress warnings for unused variables timeinfo, userData */
    (void)timeInfo;
    (void)userData;
    UINT32 i, j;
    INT32 pumpRet; //return for pump
    if (aweOS_layoutIsValid(g_AWEOSInstance))
    {
        if (aweOS_audioIsStarted(g_AWEOSInstance))
        {
            for (i = 0; i < hwInputChannels; i++)
            {
                aweOS_audioImportSamples(g_AWEOSInstance, (void*)(((UINT32*)paInputBuffer) + i), hwInputChannels, i, Sample32bit); //import samples for each input channel in the loop
            }
            
            pumpRet = aweOS_audioPumpAll(g_AWEOSInstance);

            for (j = 0; j < hwOutputChannels; j++)
            {
                aweOS_audioExportSamples(g_AWEOSInstance, (void*)(((UINT32*)paOutputBuffer) + j), hwOutputChannels, j, Sample32bit); //export samples for each output channel in the loop
            }

            if (pumpRet < 0)
            {
                printf("USER APP DEBUG: pump failing with error code %d \n", pumpRet);
            }
        }
    }
    
    if (statusFlags != 0)
    {
        printf("Error in PortAudio Stream, statusFlags != 0. StatusFlags == %d, %x \n", (INT32)statusFlags, (INT32)statusFlags);
    }

    return paContinue;
}
```

### Control Interface

Module variables can be controlled from the application in both products using the control interface variables defined in the ‘\_ControlInterface\*.h’ files generated in Designer. AWELib uses the ‘\_ControlInterface\_AWE6.h’ files, and AWE Core OS uses ‘\_ControlInterface.h’.

AWELib has separate functions for setting and getting module variable arrays, and scalar values. AWE Core OS instead has a single function for set and get, and always requires the size of the variable to be passed in \(size of 1 for all scalars\).

**AWELib**

The ID and OFFSET values used below are defined in the ‘\_ControlInterface\_AWE6.h’ file for a layout with a Scaler module called Scaler1, and a Meter module called Meter1.

```cpp
#include “passthrough_ControlInterface_AWE6.h”

...

// Set Scaler1 gin attributes
Sample scaler1Gain, scaler1TargetGain;
Sample meter1Values[AWE_Meter1_value_SIZE];
INT32 error;

scaler1Gain.fVal = -10.0;
pAwelib->SetValue(MAKE_ADDRESS(AWE_Scaler1_ID, AWE_Scaler1_gain_OFFSET), scaler1Gain);

scaler1Gain = pAwelib->FetchValue(MAKE_ADDRESS(AWE_Scaler1_ID, AWE_Scaler1_gain_OFFSET), &error);

pAwelib->FetchValues(MAKE_ADDRESS(AWE_Meter1_ID, AWE_Meter1_value_OFFSET), AWE_Meter1_value_SIZE, meter1Values);
```

**AWE Core OS**

In addition to the unified get/set control interface API’s, AWE Core OS also provides a new API to check if the module exists in the layout. This can be used to protect the application from accessing modules that do not exist, which can result in undefined behavior. The HANDLE and SIZE values used below are defined in the ‘\_ControlInterface.h’ file.

```cpp
#include "passthrough_ControlInterface.h"

…

ret = aweOS_ctrlGetModuleClass(g_AWEOSInstance, AWE_Scaler1_gain_HANDLE, &classId);
if (E_SUCCESS == ret)
{
        // Check that module assigned this object ID is of module class SinkInt
        if (AWE_Scaler1_classID == classId )
        {
                //Set the gain to -10 dB
                Float scaler1Gain = -10.0;
                aweOS_ctrlSetValue(g_AWEOSInstance, AWE_Scaler1_gain_HANDLE, ( void *)&scaler1Gain, 0, AWE_Scaler1_gain_SIZE);
        }
}
```

### Miscellaneous

See the AWE Core OS Integration Guide and API doc for new features that do not exist in AWELib like Tuning logs, wav file recording, profiling enable/disable, etc.

#### 

