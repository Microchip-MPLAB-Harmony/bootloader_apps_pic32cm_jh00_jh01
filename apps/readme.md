---
parent: UART-I2C Factory Programmed Bootloader Applications
title: UART-I2C Factory Programmed Bootloader
has_children: false
has_toc: false
nav_order: 1
---

[![MCHP](https://www.microchip.com/ResourcePackages/Microchip/assets/dist/images/logo.png)](https://www.microchip.com)

To clone or download this application from Github,go to the [main page of this repository](https://github.com/Microchip-MPLAB-Harmony/bootloader_apps_pic32cm_jh00_jh01) and then click Clone button to clone this repo or download as zip file. This content can also be download using content manager by following [these instructions](https://github.com/Microchip-MPLAB-Harmony/contentmanager/wiki)

# UART-I2C Factory Programed Bootloader

This example application shows how to use the UART-I2C Factory Programed Bootloader for PIC32CM JH00/JH01.

### Bootloader Application

Key Features
- The Bootloader is programmed at the Flash memory location (0x0000-0000) and takes up to 4KB of Flash
memory. It is protected by default using the BOOTPROT fuse setting from any erase-writes. The BOOTPROT
value is set to 4KB (0x1000).
- The Bootloader startup code copies the Bootloader from the Flash memory to the SRAM from (0x2000-0010)
before calling the main function. The main function (and therefore the entire Bootloader) runs from the SRAM to
allow upgrading of the Bootloader code itself in Flash memory.
- The Bootloader provides a feature to read the current version of the Bootloader running by using the read
version command
- The Bootloader provides a feature to program the Fuse settings using a separate command
- The Bootloader will provide two ways to trigger the Bootloader mode:

    External trigger: Bootloader will be triggered based on the status of a GPIO pin
    
    Internal trigger: Bootloader will be triggered based on the trigger pattern written at a specific location in SRAM by the application firmware
    
- After a new application image is programmed, the Bootloader will verify the programmed application space by
generating a CRC-32 value and comparing it with the CRC-32 received from the host. The application CRC will
not be verified after every reset before jumping to the application space for faster startup.
- The Bootloader will read the first four bytes of application space (0x1000) to decide if a valid application is
present. If the contents of the first four bytes are not 0xFFFF-FFFF, then the Bootloader assumes a valid
application is present and jumps to the application. If a valid application is not present, then the Bootloader will
wait, and remain in Bootloader mode.

Key Requirements

- By default the Bootloader expects the application to start from the 0x1000 location. Therefore, the application
should be built to start from the 0x1000 Flash location.
- In order to update the Bootloader code, the user should first set the BOOTPROT fuse to 0xF to disable the write
protection, then send the new Bootloader binary
- External Pull-ups need to be used for the I2C SDA and SCL lines

#### UART-I2C Bootloader Memory Layout

![uart_i2c_bootloader_memory_layout](docs/images/uart_i2c_bootloader_memory_layout.png)

#### UART-I2C Bootloader System Level Execution Flow

![uart_i2c_bootloader_system_execution_flow](docs/images/uart_i2c_bootloader_system_execution_flow.png)

#### UART I2C and Boot Pin Configurations

| Protocol  | Port Pin Configuration                                     | SERCOM Instance  |
| ----------|:-----------------------------------------------------------| -----------------|
| UART      | PA18 - TX - SERCOM1/PAD2                                   |     SERCOM 1     |
|           | PA19 - RX - SERCOM1/PAD3                                   |                  |
| I2C       | PA22 - SDA - SERCOM3/PAD0 - External Pull-up required      |     SERCOM 3     |
|           | PA23 - SCL - SERCOM3/PAD1 - External Pull-up required      |                  |
| Boot pin  | PA27 - GPIO - Input - Pull-up enabled                      |         -        |

#### UART and I2C communication configurations

| Protocol  | Communication Parameters                                   |
| ----------|:-----------------------------------------------------------|
| UART      | 115200 baud, 8-bit, Even parity, 1 Stop bit                |
| I2C       | Up to 400 kHz speed.                                       |
|           | I2C client 7-bit address (0x40) - 1000-000x (where x = 0 for write, 1 for read). |
|           | Address is fixed and not programmable through any hardware setting.      |

### Bootloader Entry Mechanisms
#### Default Entry Mechanism:

- The Bootloader will run automatically if there is no valid application firmware. Firmware is considered valid if the
first word at the application start address is not 0xFFFFFFFF.
- Normally this word contains the initial stack pointer value, so it will never be 0xFFFFFFFF unless device is erased

#### Trigger Entry Mechanism:

- External trigger:

  – The Boot pin should be used to trigger the Bootloader after reset (See the following table)

  – Drive the respective Boot pin to a low state, then reset the device. Upon reset, the Bootloader runs and checks the status of the Boot pin. If the status of the pin is low, the Bootloader enters the firmware upgrade mode.

- Bootloader Trigger Pattern in SRAM:

  – The application can trigger the Bootloader by writing a Bootloader trigger pattern to the first four words (1 word = 4 bytes) of SRAM (0x20000000), then resetting the device

  – Upon reset, the Bootloader runs and checks the first four words of the SRAM for the presence of the Bootloader trigger pattern. If found, the Bootloader enters the firmware upgrade mode.

  – To invoke the Bootloader, the trigger pattern is 0x5048434D

| Trigger mode          | Trigger method                                                                    |
| ----------------------|-----------------------------------------------------------------------------------|
| Boot pin              | PA27 = Active low                                                                 |
|                       | (Internal pull-up is enabled. No hardware/software de-bouncing is implemented.)   |
| SRAM trigger pattern  | SRAM Word[0] = 0x5048434D                                                         |
| (0x20000000)          | SRAM Word[1] = 0x5048434D                                                         |
|                       | SRAM Word[2] = 0x5048434D                                                         |
|                       | SRAM Word[3] = 0x5048434D                                                         |

### Fuse Configurations
- The Bootloader project will have the Fuse settings set to default except the BOOTPROT fuse

  BOOTPROT is set to 4096 Bytes (0xB) during production to protect the Bootloader from accidental erase/writes.
  
- The Fuse settings can be changed in following ways:

  The Bootloader provides a separate command to program the Fuse Bits received from the host. Refer to the Bootloader protocol documentation and respective host utility documentation for more information.
  
  The application can check the Fuse bits during the boot-up and then program any change required using the NVMCTRL peripheral
  
- As fuse settings are placed in a higher memory location in the Flash (User Row), the Fuse settings need to be disabled for the application project, which will be boot-loaded since the size of the binary file becomes too large when the fuse settings are enabled

- When updating the Bootloader itself, make sure that the fuse settings for the Bootloader application are also
disabled


### UART Host Tool
A python host utility is provided as part of the MPLAB Harmony v3 Bootloader package which can be used to communicate with the Bootloader to send a binary over the UART from the Host PC.

Refer to [UART Bootloader Host Script Help](https://microchip-mplab-harmony.github.io/bootloader/?GUID-8BE0388C-8563-4ED8-9C17-F3FE7B88FE51) for more details on usage of the host utility.

### I2C Embedded Host
The MPLAB Harmony v3 based I2C embedded host application is provided as a reference which sends a binary over the I2C from a host microcontroller.
Refer to the [I2C Host App SD-Card application](docs/i2c_sdcard_host_app.md) for more details.

### Other References
- Refer to the [Bootloader Library](https://microchip-mplab-harmony.github.io/bootloader/) for understanding:

  Bootloader framework
  
  How the Bootloader library works
  
  Bootloader library configurations
  
  Bootloader memory layout
  
- Refer to the [UART Bootloader Protocol](https://microchip-mplab-harmony.github.io/bootloader/?GUID-8828D474-F227-4FE0-88EE-135AA591750F) for understanding:

  Details on protocol
  
  Various commands supported
  
- Refer to the [I2C Bootloader Protocol](https://microchip-mplab-harmony.github.io/bootloader/?GUID-3942704D-13D2-4E8E-A9DC-4055E7F6D5AB) for understanding:

  Details on protocol
  
  Various commands supported
