# Wake Word Engine and ASR Integration for AWECoreOS

This application note describes how Wake Work Engine and ASR commands work with Audio Weaver. The first section of this document contains information about VR \(Voice Recognition\) State, why we’d need it and details of its implementation. The second section contains information about the implementation of the VR State in the TalkToSample App.

## **VR State**

### **What is a VR State?**

The VR\( Voice Recognition\) State is information from the cloud to the AWECoreOS / AWECore instance and tells it to listen to the ASR commands. As soon as the cloud notifies the AWECoreOS/AWECore instance, the value of the VR State changes to `0/1`.

### **Why do we need a VR State**?

VR State is needed to make sure that important speech content is not cancelled in the interference cancellation algorithm.

As soon as the cloud notifies the AWECoreOS instance to listen to the ASR commands, the Interference Cancellation and DOA\( Direction of Arrival\) algorithms stop adapting to incoming speech and store the latest filter coefficients. This is especially important in the case where the noise levels and speech levels are almost the same. In that case, the DOA algorithm could point to the speech direction as noise, thereby eliminating the speech in the Interference Cancellation.

### **Implementation of the VRState**

Figure 1 below shows the implementation of the VR State. As can be seen in the figure, the VR State becomes active as soon as the Trigger Word is detected. When either of the Wake Word Engines are triggered, the application code should set the VR State to `1` and ignore the other WWE to eliminate double triggers.

Once the VR State becomes active, it listens to the ASR commands. In cloud-based systems, the information on the end of utterance is provided. As soon as the end of utterance information is provided to the AWECoreOS / AWECore instance, the VR State becomes inactive.

In the TalkTo Sample App, the VR State becomes `0` once a timeout is reached. However, the most efficient way to implement the VR State is through notification of end of utterance, which is the case in cloud-based systems.

## **Implementation of the VR State in TalkTo Sample App:**

### **Basic Introduction of TTSA**

#### **Initialization of AWECoreOS Instances**

* In the TalkTo Sample App, two instances of Audio Weaver processor are instantiated- VUI processor and Playback Processor. The VUI processor processes the incoming microphone data before feeding it to the Sensory Phrase Spotters.
* The Playback Processor is loaded with the songs that are sent to the audio output.
* For more information, check the README file in TalkTo Sample App.

### **WWE and ASR**

#### **Two Sensory phrase spotters are instantiated.**

* One of them are loaded with a wake word model that responds to the phrase `Alexa`
* The second spotter is loaded with an ASR model that responds to the following phrases: `play music`, `stop music`, `pause music`, `next song`, `previous song`

#### **Sensory phrase spotter instances are created by the constructor of the TalkToSampleApp class.**

* The code below creates the first instance of Sensory:

```cpp
trigger0Spotter = std::make_unique<SensoryPhraseSpotter>(config.trigger0SpotterModelFilename, 
                                                            config.trigger0SpotterSensitivity,
                                                            trigger0SpotterCallbackFunction, 
                                                            this);
```

* The following code creates the second instance of the Sensory:

```cpp
commandSpotter = std::make_unique<SensoryPhraseSpotter>(config.commandSpotterModelFilename, 
                                                        config.commandSpotterSensitivity,
                                                        commandCallbackFunction, 
                                                        this);
```

**Audio Data:**

* The `aweOS_audioPumpAll` function shown below takes the incoming microphone audio data in the input buffer, processes it and the output buffer is fed to the WWE and Command Spotter Engines.

```cpp
error = aweOS_audioPumpAll(vuiProcessor);
```

* The code below shows how we feed the audio data to the Sensory engine. The two buffers- `trigger0SpotterBuffer` and `commandSpotterBuffer` are fed with the outputs of the `aweOS_audioPumpAll` function and the sensory engines process these buffers to spot ASR commands or Wake Words .

```cpp
    error = trigger0Spotter->process(trigger0SpotterBuffer, (int)framesPerBuffer / config.decimationRatio);
    throw_assert(error == E_SUCCESS, "trigger0Spotter->process() -> " << error);
    error = commandSpotter->process(commandSpotterBuffer, (int) framesPerBuffer / config.decimationRatio);
    throw_assert(error == 0, "commandSpotter->process()");
```

### **Implementation of the VR State**

#### **Trigger Spotter Callback functions and implementation of Voice Recognition State \(VRState\):**

* As shown in the code below, the callback functions is triggered when a keyword is detected.

```cpp
int TalkToSampleApp::trigger0SpotterCallback(const std::string &phrase, double begin, double end) {
  // This function sets the timeout to config.vrStateTimeout
   vrStateGenerator->signalWakeWordDetection(); 
   return 0;
}
```

{% hint style="info" %}
A timer is used to prevent double triggering of the wake word.
{% endhint %}

* As soon as the wake word is detected, a timeout value is set , the VR State becomes active, and the Playback Processor ducks playback. The VR State remains active until a timeout value is reached or an ASR command is detected.
* In the TalkToSampleApp, the timeout value is set for 180 blocks. At a block size of 768 and sampling rate of 48000Hz, this corresponds to a timeout value of 2.88 seconds.
* If no ASR command is detected for 2.88 seconds after the detection of the wake word, the VR state becomes inactive.

#### **Command Spotter Callback functions and implementation of VR State:**

* The Command Spotter Callback function shown below is triggered when an ASR command is spotted.

```cpp
int TalkToSampleApp::commandSpotterCallback(const std::string &phrase, double begin, double end) {
……
    vrStateGenerator->signalCommandDetection();
    if (songProvider->isSongPlaying()) {
       std::cout << "Playing song " << songProvider->getCurrentSong() << "\n";
     }
else {
   std::cout << "Playback stopped" << "\n";
 }

    return 0;
}
```

* As soon as the ASR command is detected, the timeout value is set to 0, thereby making the VRState inactive.

#### **Audio Callback and VR States:**

* The function `audioCallback` shown below reads the current VR State and compares it with the previous VR State.
* If there is a change in the VR State, the updated VR State is then displayed on the screen.
* When the VR State changes, the AWECoreOS instance of the VUI processor is notified.

```cpp
AudioStream::CallbackResult
TalkToSampleApp::audioCallback(const void *hardwareCaptureBuffer, void *hardwarePlaybackBuffer,
                               unsigned long framesPerBuffer, AudioStream::CallbackFlag statusFlag) {

    unsigned int vrStatePrevious = vrState;
    vrState = vrStateGenerator->getState();

    //check the status
    if (vrStatePrevious != vrState) {
        std::cout << "Setting AWE_VRState to " << vrState << std::endl;
        error = aweOS_ctrlSetValue(vuiProcessor, AWE_VRState_value_HANDLE, &vrState,
                                0, AWE_VRState_value_SIZE);
        if (E_SUCCESS == error) {
        std::cout << "[vuiProcessor] VRState SetValue:" << vrState
                    << " Error:" << error << std::endl;
        } else {
        std::cout << "[vuiProcessor] SetValue error " << error << " "
                    << aweOS_errorToString(error) << std::endl;
        }

        error = aweOS_ctrlSetValue(playbackProcessor, AWE_duckPlayback_value_HANDLE,
                                &vrState, 0, AWE_duckPlayback_value_SIZE);
        if (E_SUCCESS == error) {
        std::cout << "[playbackProcessor] DuckPlayback SetValue:" << vrState
                    << " Error:" << error << std::endl;
        } else {
        std::cout << "[playbackProcessor] SetValue error " << error << " "
                    << aweOS_errorToString(error) << std::endl;
        }
    }
```

