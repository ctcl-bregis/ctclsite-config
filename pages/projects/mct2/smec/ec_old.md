## Introduction
The embedded controller manages various subsystems on MediaCow Touch 2 and runs independently of the LattePanda Mu and the operating system.

Until July 30, 2024, I planned to use two microcontrollers named "SMEC" and "PMEC" for System Management and Power Management, respectively. Now, I plan to use just one microcontroller. The name "SMEC" was reused for the one microcontroller on the carrier board. Hereinafter, the embedded controller would be refered to as just SMEC.

## Microcontroller Selection
Up to September 11, 2024, the embedded controller would have consisted of a single STMicroelectonics STM32L496. Now, a Sunplus Technology/Tibbo Technology SP7021 SIP would be used as the embedded controller. It may seem like I have completely lost my mind but there are some good reasons for me to use the SP7021.

## Power Management
Power Management has became a major concern with the switch to the SP7021.

### Peripheral Usage 
Many of the interfaces of the SP7021 will be disabled as they are not used. This would save a considerable amount of power. This includes HDMI, Ethernet, USB.

### CPU Power Management
It is likely that I would not need more than one ARM Cortex-A7 core along with the ARM926 core. 

## Functions

### OLED display
The OLED display recently introduced to the design would be driven by SMEC. SMEC uses the Intel 8080 parallel interface for communication with the OLED display. The parallel interface can be used with the STM32's FMC (Flexible Memory Controller) feature.

### Button Pad
The button pad, outlined in the [OLED Display and Button Panel](../oled/) page.

### Battery Management
When the system is powered on or off, SMEC would constantly read data from the battery pack, USB PD IC and charger IC. The latter two likely not as often as they are just used during charging. 

### USB PD management
The critical functions of USB Power Delivery is handled by the TPS65988. SMEC communicates with this chip for port status and control. 

SMEC controls the two buck-boost regulators for USB source power. The TPS65988 would communicate with SMEC to set the output voltages of the buck-boost regulators.


## IO Usage 
This is the IO used on SMEC.

### I2C
Three I2C interfaces are used

Following device addresses are 7-bit. 

#### Bus 1
Bus 1 shall always be online and is used for communicating with various SMBus devices.

- SDA - SYS_SMB_SDA
- SCL - SYS_SMB_SCL
- SMBus Alert - SYS_SMB_ALT

Ordered by address, increasing:

- Battery pack fuel gauge - TBD
- TPS65988 Type-C PD controller - 0x20
- TCA8418 Keypad Controller - 0x34
- I210AT Ethernet Controller - 0x49
- LTC4162-LAD battery charge controller - 0x68

#### Bus 2
Bus 2 is dedicated to communication between SMEC and the TCP controllers on the Intel N100. 

Label names:
- SDA - SML_SDA
- SCL - SML_SCL
- SMBus Alert - SML_ALT

Addresses:
- TCP0 - 0x??
- TCP1 - 0x??

#### Bus 3
Bus 3 is dedicated to communication between the OS and SMEC.

Label names:
- SDA - SOM_SMB_SDA
- SCL - SOM_SMB_SCL
- SMBus Alert - SOM_SMB_ALT

Addresses: 
- LattePanda Mu SoM - TBD

#### Bus 4
Bus 4 may be used for communication with I2C devices that do not support SMBus.

- MP8859 Buck-Boost converter for TCP0 source - 0x60
- MP8859 Buck-Boost converter for TCP1 source - 0x66
- ICM-20948 ICM - 0x69
- IS31FL3193 LED controller - 0x6B
- BMP384 Digital pressure sensor - 0x76

### GPIO

Input: Input to SMEC from device
Output: Output from SMEC to device

This section is split into subsections due to the amount of connections required

#### Power Management Control

- USB Load Switches
  - EUSB0_EN - Output
  - EUSB0_FLT - Input
  - EUSB1_EN - Output
  - EUSB1_FLT - Input
  - EUSB2_EN - Output
  - EUSB2_FLT - Input
  - EUSB3_EN - Output
  - EUSB3_FLT - Input
- USB_5V Regulator
  - USB_5V_EN - Output
  - USB_5V_PG - Input
- VSYS_SOM Power Switch
  - VSYS_SW_IN - Output
  - VSYS_SW_DEN - Output
  - VSYS_SW_IS - Analog Input
- TCP_PPHV1 Regulator
  - TCP_PPHV1_EN - Output
- TCP_PPHV2 Regulator
  - TCP_PPHV2_EN - Output
- VSYS_LV Regulator
  - VSYS_LV_PG - Input
- USB_5V Regulator
  - USB_5V_PG - Input
- OLED_VDD Regulator
  - OLED_VDD_EN - Output
- OLED_VOLED Regulator
  - OLED_VOLED_EN - Output

#### Device Control

- Keypad
  - KP_RST - Output
  - KP_SW_RAD - Input
  - KP_SW_CAM - Input
- HDMI Companion IC
  - HDMI_CT_HPD - Output
  - HDMI_LS_OE - Output
- Audio CODEC 
  - HDA_RST - Output
  - HDA_PB - Output
  - 
- System on Module (LattePanda Mu)
  - PROCHOT - Input
  - PWRBTN - Output
  - RSTBTN - Output
  - SLS_S0 - Input
  - SLS_S3 - Input
  - USB_OC (SOM_USB_OC) - Output
- Ethernet
  - LAN_EN - Output
  - LAN_PG - Output
- M.2 Key E
  - M2E_WDISABLE1 - Output
  - M2E_WDISABLE2 - Output
- LED Indicator
  - LED_VBM - Input
  - LED_EN - Output

### Flexible Memory Controller
The STM32 FMC interface is used by the OLED display on the side of the case. 

### UART
One or more UART serial interfaces of SMEC may be used. 

#### SIO_UART_RX/TX
SIO_UART is the serial connection between SMEC and the LattePanda Mu's on-board Super IO controller.

## Power
*See [Battery and Power Management](../power/) page for details on system power management*

It is critical that SMEC uses very little power, especially when the system is powered off. 

## System Communication
As mentioned above, there are two interfaces that SMEC can use to communicate with the OS: SMBus and USB.

It is expected that a custom daemon or Linux kernel module would have to be written for the system to make use of SMEC. I would not implement driver support for Microsoft Windows (but, of course, the project is open source so one can add support for Windows if they really wanted to).

This daemon or driver would send ACPI events for other daemons such as [acpid](https://linux.die.net/man/8/acpid) to make use of. 

## Software
As of August 21, 2024, I have not decided if SMEC would use FreeRTOS or the firmware would run "bare metal".

## Misc
This section covers miscellaneous details about the embedded controller.

### Clock Source

#### Oscillators
Instead of using a conventional crystal oscillator, SMEC would make use of MEMS oscillators. Using this kind of oscillator has the benefits of MEMS-based oscillators along with easier implementation since they just need a single passive component being the power supply decoupling capacitor. I have had an interest in MEMS oscillator technology since late 2018 and this is not the first time I have used them in a design.

Currently, there are two external oscillators used for SMEC:
- System: 27MHz SiTime SiT8008BI-11-33E-27.000000G
- RTC: 32.768KHz SiTime SiT1533AI-H4-DCC-32.768

#### STM32 Clock Configuration
The 48MHz clock input directly goes to the ARM core clock (SYSCLK), resulting in SMEC running at a clock speed of 48MHz instead of advertised maximum of 80MHz. As seen with the [Flipper Zero that runs its STM32WB55 at 64MHz (Cortex-M4 core)](https://docs.flipper.net/development/hardware/tech-specs), 48MHz should be plenty for what is planned to be done with SMEC.

The 32.768KHz LSE clock input directly goes to the RTC (Real-Time Clock)

