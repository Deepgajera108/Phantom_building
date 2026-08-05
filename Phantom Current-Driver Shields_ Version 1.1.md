## Phantom Current-Driver Shields: Version 1.1

### Initial Assembly

As part of the MEG biosignal phantom project, I was responsible for assembling and testing the Arduino-based current-driver shields used to activate the phantom sources. The required components, PCB files, and general assembly information were available through the MEG Biosignal Phantom GitHub repository:

https://github.com/tbardouille/MEG_biosignal_phantom/tree/main/Contributors/Donders

The phantom consists of current-dipole sources and Head Position Indicator (HPI) coils connected through twisted wires to an external current-driver circuit. The driver produces time-varying currents that activate the sources sequentially, allowing the resulting magnetic fields to be measured by the MEG system.

I used reference photographs available through the GitHub repository and its issue discussions to identify the correct component orientation and placement. Following these references, I assembled two version 1.1 Arduino current-driver shields.

**Figure 9.** The two version 1.1 Arduino current-driver shields assembled during this project.

**Figure 10.** Reference image used to verify component placement and shield assembly.

### Identifying the Correct Arduino Driver Code

After completing the hardware assembly, I needed Arduino code to operate and test the shields. 

I used this code for Arduino Uno/Leonardo implementation:

https://github.com/robertoostenveld/arduino/tree/main/uno_dac7578

This code is designed to communicate with the DAC7578 boards and can automatically detect whether one, two, or three shields are connected to the Arduino.

During this discussion, I was also warned about an important difference between PCB versions 1.0 and 1.1. In version 1.0, the DAC7578 footprint was accidentally reversed in the PCB design. Consequently, the DAC7578 had to be soldered upside down for the circuit to operate correctly. Installing it in the normal orientation and powering the board could permanently damage the DAC7578.

I confirmed that the PCBs I had assembled were version 1.1, in which this design error had been corrected. This verification was important before applying power to the completed boards.

### Software Dependencies and Initial Testing

During compilation of the Arduino program, I found that several additional libraries were required. These included the `Ticker` library:

```cpp
#include <Ticker.h>
```

The Ticker library is available from:

https://github.com/sstaub/Ticker

The appropriate DAC7578 library was also required so that the Arduino could communicate with the digital-to-analog converters through the I²C interface.

After installing the required libraries and resolving the compilation dependencies, I successfully uploaded the program to the Arduino Uno R3. The program ran, and I was able to begin testing the two connected shields.

### Two-Shield I²C Address Collision

Although both shields were physically connected, the Arduino did not identify them as two independent devices. The serial output suggested that both shields were being detected as the same shield. When I measured the output signals, the corresponding channels on the two shields produced the same currents and frequencies.

This indicated that both DAC7578 devices were responding to the same I²C address. Because both devices had identical addresses, the Arduino commands intended for one shield were simultaneously being received by the other shield.

I documented the problem and asked for guidance through GitHub issue number 6:

https://github.com/tbardouille/MEG_biosignal_phantom/issues/6

The response explained that one of the address contacts on the underside of the second DAC7578 board had to be soldered to change its I²C address. The relevant instructions were available from Adafruit:

https://learn.adafruit.com/adafruit-dac7578-8-x-channel-12-bit-i2c-dac?view=all#i2c-address-jumper-3193263

### Configuring the DAC7578 I²C Address

The DAC7578 board uses three possible I²C addresses, depending on the configuration of its address jumper:

- With none of the address jumper pads connected, the address pin is left floating and the default I²C address is **0x4C**.
- Connecting the centre jumper pad to the pad on its left connects the address pin to ground and changes the address to **0x48**.
- Connecting the centre jumper pad to the pad on its right connects the address pin to the supply voltage and changes the address to **0x4A**.

Both shields had initially been left at the default address of 0x4C. Therefore, the Arduino could not distinguish between them.

To correct the problem, I removed the second shield from the assembled stack and accessed the address jumper on the underside of its DAC7578 board. I then soldered the centre address pad to one of the adjacent pads, assigning the second DAC7578 a different I²C address.

After changing the address and reconnecting the shield, the Arduino correctly identified the two DAC7578 devices as separate boards. Commands could then be sent independently to the channels on each shield, and the two boards generated their intended currents and frequencies without duplicating one another.

**Figure 11.** Arduino serial output obtained when both DAC7578 shields were configured with the same default I²C address.

**Figure 12.** Address-selection jumper on the underside of the DAC7578 board. Connecting the centre pad to either adjacent pad assigns the board a different I²C address.

This was a simple modification, but it would have been easier to complete before soldering and stacking the full circuit. This experience showed the importance of assigning unique I²C addresses before assembling multiple identical devices on the same communication bus.

### Phantom Connection Cable and Junction Wiring

In parallel with the driver-shield work, I constructed the cable required to connect the driver electronics to the phantom head. The cable contains the connections required to carry the activation signals from the current-driver shields to the individual phantom sources.

Care was required to maintain consistent channel numbering between:

1. The Arduino output channels;
2. The DAC7578 shield channels;
3. The connecting cable;
4. The junction circuit; and
5. The corresponding phantom sources.

The wires were labelled and checked for continuity to ensure that each electronic output was connected to the intended phantom source. I also considered the circuit required at the phantom end to distribute the signals correctly and provide a secure, detachable connection between the driver and the phantom.

**Figure 13.** Completed cable used to connect the Arduino current-driver system to the phantom.

### Final Testing and Outcome

After changing the I²C address of the second DAC7578 shield, completing the connection cable, and checking the wiring, I tested the complete system.

Both version 1.1 shields were successfully detected by the Arduino and could be controlled independently. The channels produced the intended signals, and the cable correctly transferred those signals to the phantom sources.

The phantom driver is now functioning correctly, and the completed phantom can generate controlled magnetic signals for subsequent OPM-MEG testing and localization experiments.

### Key Outcomes and Lessons Learned

The main outcomes of this work were:

- I gained practical experience assembling and soldering electronic PCBs.
- I assembled two Arduino current-driver shields using PCB version 1.1.
- I identified and installed the correct Arduino Uno DAC7578 driver code.
- I installed and configured the required Arduino libraries.
- I verified the importance of distinguishing PCB version 1.1 from the defective version 1.0 design.
- I diagnosed an I²C address collision between two DAC7578 boards.
- I configured a unique I²C address for the second shield using the address jumper.
- I constructed and tested the cable connecting the driver system to the phantom.
- I successfully operated the completed phantom using two independently controlled current-driver shields.

An important practical lesson was that I²C addresses should be configured before the shields are completely assembled. Changing the address jumper is straightforward when the DAC7578 board is accessible, but becomes more difficult after the board has been soldered and incorporated into the complete shield assembly. Nevertheless, troubleshooting this issue improved my understanding of I²C communication, hardware addressing, PCB assembly, and the overall operation of the phantom current-driver system.
