## Battery
A replaceable battery pack is used to power the system. 

### Details
The battery pack in particular is one intended as a replacement battery for laptops that use HP FM08 battery packs. A battery intended for an existing laptop is used to save costs while still providing SMBus communication with the in-pack battery management IC including the fuel gauge, which is a crucial part of the battery power subsystem. 

As of September 25, 2024, the pinout of the battery pack is still not completely known and not much testing has been done. 

## USB Type-C Power Delivery

Feature overview:
- 2x USB Type-C
- DisplayPort alternate mode support on both connectors
- 5-20v sink and source capabilities
- Status readable through both the system and SMEC side display

DisplayPort and USB 3.0 signal multiplex is already handled by the Intel N100 SoC itself so no multiplexer ICs are required on the carrier board. The USB PD circuit is likely the most complex part of the circuit design, more so than SMEC itself.

### PD Controller
The PD controller chosen for the device design is the Texas Instruments TPS65988. The IC allows for dual-port sink and source capabilities. 

### HV Source Voltage
Two buck-boost converters are on the PPHV supplies of the PD controller. This allows for PD source capabilities with voltages up to 20v while plugged in and on battery.

Originally, two Monolithic Power Systems MP8859 Buck-Boost converters were used up to September 25, 2024, when the two MP8859 were switched out for two Texas Instruments TPS55189-Q1 due to similar cost while supporting a higher output current.

An option to have USB PD sources available while the system is powered off may be available. This is inspired by a similar function found on the HP ZBook Studio G3 that I currently make use of where 5V power over the USB Type-C ports is available while the system is offline with a configurable minimum required battery percentage.



