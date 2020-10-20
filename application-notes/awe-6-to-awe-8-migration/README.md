# AWE 6 to AWE 8 Migration

The legacy AWE 6 product line, including AWECore for embedded/bare metal targets, and AWELib for rich OS targets \(Linux, Android, Windows\), has been in use for almost 4 years. AWE Core 8 is now replacing AWE Core 6, and AWE Core OS is replacing AWELib.

### But AWE 6 works well for me, why should I switch?

The main goal of the updates in AWE 8 is to make integrations of the AWE Core easier on any customer platform. To this end, improvements were made to the product in the following ways:

* Smaller, more intuitive API
  * AWE6 API functions spread across multiple, large header files, making it difficult to know which functions to use. AWE8 has a single API header, “AWECore.h”
  * Consistent naming conventions.
    * All AWE8 API’s start with ‘awe\_’, and functions are further categorized into subgroups. For example, audio related functions begin with ‘awe\_audio’.
* Multi-core applications easier to implement
  * Removed the CoreDescriptor/callback function for packet processing and routing packets. In AWE 8, BSP author receives core packets and then has full freedom to handle them as needed
* Enhanced documentation
  * Doxygen generated .pdf and .html, fully navigable, and with more example apps
* Built in support for common audio data types and built in channel matching
* More available modules

AWELib -&gt; AWE Core OS

![](../../.gitbook/assets/assets_knowledge-base_-miuj5cak-sr92jfdwfs_-miujauszfxagukutywe_0.jpeg)

In addition to the above, AWE Core OS also provides the following improvements over AWELib

* Easier and better multi-rate processing capabilities \(auto-multithreading on SMP systems\)
* Simpler and more configurable instance initialization
* Integrated tuning interface for any TCP/IP enabled device
* Built in input and output audio recording to .wav files
* More and better sample applications, including a real-time audio sample app using Alsa
* Improved configurability of included modules
* C library, not C++, so more portable
* Single function to load encrypted/unencrypted layouts

