# FretNought

**Team Number:** 10

**Team Name:** Guitar Heroes

**GitHub Repository URL:** https://github.com/ese5160/a11g-final-submission-s26-s26-t10-guitar-heroes

| Team Member Name  | Email Address | GitHub Handle |
| ----------------- | ------------- | ------------- |
| Matilda Dingemans | mdingema@seas | MatildaZD     |
| Sophia Fu         | sophiafu@seas | sophfu        |

## 1. Video Presentation

Link to Video:

## 2. Project Summary

FretNought is a smart guitar learning system that places addressable LED strips beneath the strings on the fretboard to show users exactly where to place their fingers for any chord, paired with a wireless smart pick that detects strumming via an onboard IMU and microphone.

**Inspiration**

- We were inspired by how guitar beginners often get frustrated because memorizing chord shapes from a chart and translating them onto a physical fretboard is frustrating, and we wanted to remove that translation step entirely.

**Internet Functionality**

- The device leverages Wi-Fi to connect to a companion mobile webpage where users input chords or full songs, and the pick streams strumming events back to the app so it can advance through chord progressions in real time.

**Device Functionality**

- The system is built around two Wi-Fi-connected SiWx917 modules: one for the fretboard unit and one in the smart pick wrist watch. The fretboard unit drives addressable LED strips routed underneath each string between frets, with a microcontroller mapping incoming chord data from the app to specific LED indices. The smart pick contains an IMU for detecting strum gestures and the fret device also has a microphone for capturing audio. When the mic picks up frequencies in the guitar's range, it sends a signal over Wi-Fi, and when the IMU detects a strum motion, it sends a separate signal. Node-RED fuses these two streams: when both events arrive within a 500 ms window, it confirms a real strum and advances to the next chord in practice mode, which filters out false positives from incidental motion or ambient sound.

The mobile web page communicates with both devices and offers two modes:

- Practice mode: user selects a chord; LEDs light up; strumming with the pick advances to the next chord.
- Performance mode: chords auto-advance based on user-set tempo and capo position for full-song playthrough.

![](Images/System_Diagram.png)

**Challenges**

The biggest hardware roadblock was a persistent Timeout 102 error when trying to flash our custom PCB. We tried nearly every fix we could find but the error kept coming back, and we were ultimately forced to migrate our firmware over to the SiWx917 dev boards in order to flash and run our code on schedule.

Once we were unblocked on flashing, the next set of challenges was writing the low-level drivers needed for our peripherals. For the microphone (I2S), we got it working by configuring the SiWx917's I2S peripheral in receive mode with the correct word size and sample rate to match the mic, then DMA'ing samples into a buffer for frequency analysis. For the IMU (I2C), we wrote a driver that handled the standard register read/write transactions over the I2C bus, including initialization of the sensor's control registers and polling-based reads of the accelerometer data. For the LED strip, we drove the addressable LEDs over SPI by carefully crafting the MOSI bit pattern and clock rate so that each SPI byte encoded the precise high/low pulse timing the LED protocol expects, letting us push full-strip color data. We were running into a problem where we could not turn the LEDs fully off, but eventually traced this to the fact that our LED update code was running inside a thread that didn't have the privileges needed to fully control the SPI peripheral's output state, so the line wasn't being driven cleanly to idle between transfers.

Our last challenge was that the metal strings kept shorting our device as the LEDs have exposed pads. We fixed this with tape and heat sink.

**Prototype Learnings**
Building FretNought reinforced how much hardware bring-up dominates an IoT project's timeline. The LED control and Wi-Fi messaging logic were straightforward compared to trying to get the custom board to a flashable state.

The project also gave us a real appreciation for how to properly route power: we put significant care into our power distribution from the start, and as a result we never ran into a single power-related issue throughout the entire build.

We also identified a few hardware mistakes we'd correct on the next iteration. Most notably, our microphone footprint was wrong. The pad layout we created didn't match the part, which we worked around with a dev board.

We'd also add more through-hole test points rather than relying on surface pads, since the pads turned out to be finicky for the permanent solder connections we needed when wiring our custom board to the dev board. Lastly, we would also add more connectors in general for testing components.

**Next Steps**
By far the most common question we got at our demo was about audio-based chord recognition. Right now the system confirms that a strum happened by fusing IMU motion with mic audio, but it doesn't verify which chord was played. Adding real chord recognition would close the loop: in practice mode, the system would only advance once the user played the correct chord.

**What did you learn in ESE5160?**
On the hardware side, we learned how to take a bunch of datasheets to a working board. We learned how to do schematic capture, footprint selection, power distribution, and effective PCB routing. On the wireless and IoT side, we got hands-on experience with Wifi, MQTT messaging, and the full OTA update flow which demystified what's actually happening when a deployed device pulls down new code. On the the cloud and backend side, standing up an Azure VM for OTA hosting and using Node-RED as the message broker and logic layer for FretNot showed us how much of an IoT system lives off of the device itself.

**Project Links**

Final Project Firmware: https://github.com/ese5160/final-project-firmware-s26-t10-guitar-heroes

Altium Folder: https://upenn-eselabs.365.altium.com/designs/folder-03D3DC70-FAF4-4D07-AA52-7B80FEADE16D

Node Red Dashboard: http://20.7.145.116:1880/dashboard

## 3. Hardware & Software Requirements

## 4. Project Photos & Screenshots

Images of Final Build:

![](Images/Img2.jpeg)
![](Images/im4.png)
![](Images/Img1.jpeg)
![](Images/img3.jpeg)

Standalone PCBA (Top):

![](Images/standalone_top.png)

Standalone PCBA (Bottom):

![](Images/standalone_bottom.png)

Thermal Camera Image (Power On):

![](Images/thermal.png)

Altium Board (2D):

![](Images/altium1.png)

Altium Board (3D):

![](Images/altium2.png)

NodeRed Dashboard (Practice Mode):

![](Images/nodered1.png)

NodeRed Dashboard (Performance Mode):

![](Images/nodered2.png)

Node Red Dashboard (Dev):

![](Images/nodered3.png)

Node Red Backend:

![](Images/nodered4.png)
![](Images/nodered5.png)

Block Diagram of System:

![](Images/System_Diagram.png)

## 5. Codebase

Final Project Firmware: https://github.com/ese5160/final-project-firmware-s26-t10-guitar-heroes

- Fret device code in folder called fret_device (supports OTA updates)
- Pick device code in folder called pick_device

Node Red Dashboard: http://20.7.145.116:1880/dashboard
