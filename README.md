# 【Hanwen】HelloWord-Smart Keyboard

![hw1](5.Docs/2.Images/hw1.jpg)

> The `Hanwen` smart keyboard is a **multifunctional** and **modular** mechanical keyboard I designed to meet my personal needs.
>
> The keyboard uses a modular design, with the **multifunctional interaction module** on the left side that can be swapped out for various custom components. The default module is a `Dynamic component` with an e-ink display and a FOC force feedback knob. The keyboard runs on firmware for both the keyboard body and modules, which I developed using ARM Cortex-M chips. The keyboard body uses a shift register for optimized key scanning circuits. Both the modules and keyboard body can be used independently or communicate with each other via a serial protocol.
>
> **This repository provides the following open-source content:**
>
> * The hardware design files for 10 PCBs used in the Hanwen keyboard, provided in the professional version of the Lichuang EDA format.
> * Structural design files for the case.
> * Source code for the main keyboard firmware (relatively complete).
> * Source code for the Dynamic component firmware (framework complete, more app extensions WIP).
> * SDK for secondary keyboard development (in development).
>
> **You can see a demonstration of the keyboard functions here:**
>
> * [【自制】我做了一把 模 块 化 机 械 键 盘 !【软核】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV19V4y1J7Hx)
>
> * [I Made A Customized Modular Keyboard ! - YouTube](https://www.youtube.com/watch?v=mGShD9ZER1c)

**Note: Issues are for project development discussions only. Do not post irrelevant messages there, as it will notify everyone who has watched the repository and cause unnecessary trouble! You can have casual discussions in the repository's Discuss section!**

---

## 1. Project Overview

### 1.0 Update Notes:
**2023.02.20 Update**
* Modified `CUSTOM_HID_EPOUT_ADDR = 2`, moved `HID_RxCpltCallback` to `CUSTOM_HID_OutEvent_FS` (the previous location could cause an issue where reports could not be sent in a while loop).

  > * Added RGB control code to the main project; you can control the keyboard’s RGB effects by sending packets via the HID protocol.
  > * Integrated with SignalRGB, added SignalRGB plugin to Software.
  > * Report packet size is 33 bytes: the first three bytes are for `reportid` (2 for this project), control command (0xAC for host control, 0xBD to disable host control), report sequence (up to 10 RGB values per packet, requiring multiple packets to combine). The remaining 30 bytes contain RGB values.


**2022.08.31 Update:**

* Added `Test-Dynamic-fw.bin` test firmware; after flashing it to the module, you can experience different force feedback effects on the knob.

  > * Note that the test firmware calibrates the motor on each power cycle; if calibration fails, you need to power cycle again (in the final version, calibration will only be required once);
  > * The two buttons on the module allow switching between modes;
  > * Hardware-wise, ensure that the FPC cable used for the module is short enough to avoid resistance and voltage drops, and first verify that the encoder works correctly (you can use Debug to check the encoder data).

**2022.08.22 Update:**

* Added 3D models in STEP format, including the entire set with the positioning plate.

**2022.08.20 Update:**

* PCB design files updated, available in the repository. All components that can be ordered directly from Lichuang have been updated to corresponding footprints for easy BOM configuration.

**2022.08.13 Update:**

* Received the new PCB samples, but due to a video release by He Tongxue this week, to avoid unnecessary pressure, I decided to delay the PCB update to next Saturday (doge).

**2022.07.31 Update:**

* Added all schematic files for the keyboard hardware (there are still some bugs in the circuit that haven’t been fixed, such as the flywires in the video, which will be updated once the new PCB samples are verified).
* Added the keyboard firmware source code.
* Added the Dynamic component source code.

### 1.1 Project File Explanation:

#### 1.1.1 Hardware

The Hardware folder contains all the circuit schematics and PCB files used in the Hanwen keyboard. The source files are provided in the [Lichuang EDA professional version](https://oshwhub.com/pengzhihui/b11afae464c54a3e8d0f77e1f92dc7b7) format, along with Gerber files that can be sent directly to manufacturers for processing.

![hw0](5.Docs/2.Images/hw5.png)

There are the following boards:

- **HelloWord-Keyboard**: The main PCB for the keyboard, with an STM32F103 controller. It can be used independently with the base, providing standard key input functionality with full independent RGB lighting.
- **HelloWord-Ctrl**: The PCB for the Dynamic component on the left side, with an STM32F405 controller. It can also be used independently with the base and provides an FOC force feedback knob, e-ink display, OLED display, and RGB lighting.
- **HelloWord-Connector**: The PCB for connecting the keyboard to the base using touch points, connected to the keyboard via an FFC cable.
- **HelloWord-Connector-Ctrl**: The PCB for connecting the Dynamic component to the base using touch points, connected via an FFC cable.
- **HelloWord-Encoder**: A magnetic encoder PCB for providing position feedback to the brushless motor. Requires a radially magnetized permanent magnet for operation.
- **HelloWord-Hub1**: PCB for extending two additional USB-A ports from the base, connected via an FFC cable and Type-C interface.
- **HelloWord-Hub2**: PCB for the additional USB-A ports on the base, currently using USB 2.0 (but can be upgraded to USB 3.0).
- **HelloWord-TypeC**: PCB for the Type-C interface on the base that connects to a computer. It includes a power management chip and a USB-HUB chip, connected to other modules via FFC cables.
- **HelloWord-OLED**: The minimal driving circuit and adapter board for the OLED screen on the Dynamic component.
- **HelloWord-TouchBar**: An optional capacitive touch strip module PCB with a 6-button linear capacitive touch sensor array, connected to the keyboard PCB via an FFC cable.

#### 1.1.2 Firmware

The Firmware folder contains the source code for all the boards mentioned above and **precompiled bin firmware** ready for flashing. It includes two main projects:

* **HelloWord-Keyboard-fw**: Firmware for the main keyboard, implementing high-speed key scanning based on hardware SPI and shift registers, bus-based RGB lighting control via hardware SPI & DMA, HID high-speed device enumeration & protocol handling, non-volatile memory configuration, and multi-layer key mapping.
* **HelloWord-Dynamic-fw**: Firmware for the Dynamic component, implementing FOC motor control, configurable haptic feedback, e-ink display driving, OLED driving, USB full-speed composite device enumeration and communication protocol, and RGB lighting control.

The projects are based on STM32HAL, and `.ioc` files are provided for generating projects in Keil or STM32IDE using STM32CubeMX. You can also compile and flash using CLion or STM32CubeIDE, as I do.

The `_Release` folder contains precompiled bin files, which can be flashed directly using software like **ST-Link Utility** or STM32CubeProgrammer.

For details on firmware implementation, refer to later sections.

> To turn CLion into an STM32 IDE, check out my tutorial: [Configuring CLion for STM32 Development【Elegant Embedded Development】](https://zhuanlan.zhihu.com/p/145801160).

#### 1.1.3 Software

The Software folder contains some desktop applications for interacting with the keyboard, such as the tool shown in the video for easily modifying the e-ink screen image. There will be future additions like **graphical key remapping software** and an **app store** for adding apps to modules (these are still under development).

#### 1.1.4 Tools

The Tools folder contains some third-party utilities, such as **STM32 ST-LINK Utility**, and driver installation tools like **Zadig**.

#### 1.1.5 3D Model

This folder contains 3D model files for all structural components of the keyboard, which can be used for 3D printing.

#### 1.1.6 Docs

Relevant reference materials, such as chip datasheets, are included here.

## 2. Hardware Architecture Explanation

**About the structural design?**

The Hanwen keyboard’s structure consists of three main parts: the **expansion dock base**, **keyboard input module**, and the **replaceable multifunctional interaction module**. The keyboard input module and replaceable interaction module connect to the top of the expansion dock base via several touch contacts:

![hw2](5.Docs/2.Images/hw2.jpg)

The keyboard body follows a standard custom keyboard stacked structure design, including damping foam, PCBA, positioning plate, and switch pads:

![hw2](5.Docs/2.Images/hw3.jpg)

The structural design of the keyboard is based on Xik
