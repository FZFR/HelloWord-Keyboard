# 【HanWen】HelloWord-Smart Keyboard

![hw1](5.Docs/2.Images/hw1.jpg)

> The `HanWen` smart keyboard is a **multi-functional**, **modular** mechanical keyboard that I designed for my own usage needs.
>
> The keyboard uses a modular design, with the **multi-functional scene interaction module** on the left side that can be replaced with various custom components. The default is a `Dynamic component` with an e-ink screen and FOC force feedback knob. The keyboard uses a keyboard firmware and module firmware I developed myself based on ARM Cortex-M chips. The keyboard body uses shift registers to implement an optimized key scanning circuit. The modules and keyboard body can be used independently or communicate with each other via serial protocol.
>
> **The open source materials in this repository include:**
>
> * Source files for the hardware design of 10 PCBs for the HanWen body, provided in LCEDA Professional file format
> * Shell design structure files  
> * Firmware source code for the keyboard body (relatively complete)
> * Firmware source code for the Dynamic component (framework completed, more APP extensions WIP)
> * Keyboard secondary development SDK (in development)
>
> **For keyboard function demonstrations, please refer to:**
>
> * [【DIY】I made a modular mechanical keyboard!【Soft Core】_bilibili](https://www.bilibili.com/video/BV19V4y1J7Hx)
>
> * [I Made A Customized Modular Keyboard ! - YouTube](https://www.youtube.com/watch?v=mGShD9ZER1c)

**Note: The Issues section is for discussing project development related topics. Please do not post meaningless messages there, as people who have watched the repository will receive notification emails and it will cause disturbance to others!!! Chatting can be discussed in the repository's Discussions section!**

---

## 1. Project Description

### 1.0 Update Notes:
**23.2.20 Update**
* Modified `CUSTOM_HID_EPOUT_ADDR = 2`, HID_RxCpltCallback position changed to CUSTOM_HID_OutEvent_FS (the original position may cause inability to send reports within the while loop).

  > * Added RGB control related code in main, can control keyboard RGB effects by sending data packets via HID protocol
  > * Integrated with SignalRGB, added SignalRGB plugin in Software
  > * Report packet size is 33 bytes, the first three bytes are reportid (2 for this project), control command (0xAC (PC control), 0xBD (disable PC control)), report packet sequence (one packet can transmit up to 10 RGB values, multiple packets need to be spliced, counting from 0); the last 30 bytes are RGB values.


**22.8.31 Update:**

* Added `Test-Dynamic-fw.bin` test firmware. After flashing to the module, you can experience various different force feedback wheel effects.

  > * Note that the test version firmware will calibrate the motor after each power-on. If calibration fails, you need to power on again (in the future, calibration only needs to be done once for official use);
  > * The two buttons on the module can switch between different modes;
  > * One thing to note about the hardware is that the module's FPC cable must be chosen to be shorter, otherwise the resistance will be too high and affect the voltage drop. At the same time, you need to confirm that the encoder is working normally first (you can use Debug to view encoder data).

**22.8.22 Update:**

* Added 3D model files in STEP format. The full set of models including the positioning plate have been released.

**22.8.20 Update:**

* PCB project updated, see the project link in the repository. All components that can be directly ordered from LCEDA have been changed to corresponding packages to facilitate BOM configuration.

**22.8.13 Update:**

* The newly sampled PCBs have been received, but due to He Tong's video release this week, in order to avoid unnecessary pressure, it was decided to delay updating the PCB project to next Saturday (doge).

**22.7.31 Update:**

* Added all design schematic files for keyboard hardware (there are still some bugs in the circuit that haven't been fixed, such as the flying wires in the video, which will be updated later after the new version PCB is sampled and verified)
* Added keyboard firmware source code
* Added Dynamic component source code

### 1.1 Project File Description:

#### 1.1.1 Hardware

The Hardware folder contains the schematics and PCB files for all the circuits used in the HanWen keyboard. Currently, source files in [LCEDA Professional](https://oshwhub.com/pengzhihui/b11afae464c54a3e8d0f77e1f92dc7b7) format and Gerber format photolithography files are provided for direct processing by manufacturers.

![hw0](5.Docs/2.Images/hw5.png)

There are several boards in total:

- **HelloWord-Keyboard**: The main keyboard PCB, controlled by STM32F103, can be used independently with the base, providing conventional key input functions, with independent RGB lights for all keys.
- **HelloWord-Ctrl**: The PCB for the Dynamic component on the left side, controlled by STM32F405, can be used independently with the base, providing FOC force feedback knob, e-ink screen display, OLED display, RGB lights and other functions.
- **HelloWord-Connector**: The contact PCB used to connect the base to the main keyboard, connected to the keyboard PCB via FFC cable.
- **HelloWord-Connector-Ctrl**: The contact PCB used to connect the base to the Dynamic component, connected to the Dynamic component PCB via FFC cable.
- **HelloWord-Encoder**: Magnetic encoder PCB, used for position feedback of the brushless motor, needs to work with a radially magnetized permanent magnet.
- **HelloWord-Hub1**: Adapter PCB for two additional USB-A interfaces expanded from the base, connected to the Type-C interface board via FFC cable.
- **HelloWord-Hub2**: Mother socket PCB for two additional USB-A interfaces expanded from the base, reserved for USB3.0 female socket and pins, but currently only using 2.0 interface, can be upgraded to USB3.0 HUB in the future.
- **HelloWord-TypeC**: Type-C interface PCB on the base used to connect to the computer, with onboard power charging management chip and USB-HUB chip, connected to other modules via FFC cable.
- **HelloWord-OLED**: Minimal drive circuit and adapter board for the OLED screen on the Dynamic component.
- **HelloWord-TouchBar**: Optional capacitive touch bar module PCB, using a 6-key capacitive touch chip to form a linear sensing array, connected to the main keyboard PCB via FFC cable.

#### 1.1.2 Firmware

Firmware provides the firmware source code for all the above boards, as well as **pre-compiled bin firmware** that can be directly flashed. It mainly contains the following two projects:

* **HelloWord-Keyboard-fw**: Firmware for the main keyboard, mainly implementing high-speed key scanning based on hardware SPI and shift registers, bus-type RGB light control based on hardware SPI & DMA, HID high-speed device keyboard enumeration & message protocol implementation, non-volatile storage configuration, multi-layer key mapping, etc.
* **HelloWord-Dynamic-fw:** Firmware for the Dynamic component, mainly implementing FOC-based motor control code, configurable haptic encapsulation class, e-ink screen driver, OLED driver, USB full-speed composite device enumeration and communication protocol, RGB light control, etc.

The projects are all implemented based on STM32HAL, so corresponding `.ioc` files are provided. You can open them with STM32CubeMX to generate corresponding keil or STM32IDE project files. Of course, you can also compile and download using CLion or STM32CubeIDE like me.

The `_Release` folder contains pre-compiled bin files that can be directly downloaded to the chip using software like **ST-Link Utility** or STM32CubeProgrammer.

Details about the firmware implementation will be explained later.

> For the method of turning CLion into an IDE for STM32 development, refer to a tutorial I posted before: [Configuring CLion for STM32 Development【Elegant Embedded Development】](https://zhuanlan.zhihu.com/p/145801160).

#### 1.1.3 Software

Software provides some PC-side host computer software for interacting with the keyboard, including the idiot-proof host computer software for modifying e-ink screen images demonstrated in the video, as well as **graphical software for modifying key positions** and application store software for **adding APPs to modules** that will be gradually supplemented later. These are still under development.

#### 1.1.4 Tools

Tools mainly provides some third-party tool software, such as **STM32 ST-LINK Utility**, **zadig** for driver installation, etc.

#### 1.1.5 3D Model

The folder contains 3D model files for all structural parts used in the keyboard, which can be used for 3D printing.

#### 1.1.6 Docs

Related reference files, including chip datasheets, etc.

## 2. Hardware Architecture Description

**About the structural design?**

The structure of HanWen includes three main parts: **expansion dock base**, **keyboard input module** and **replaceable multi-functional interaction module**. The keyboard input module and replaceable multi-functional interaction module are connected to the top of the expansion dock base through several contact-type contacts:

![hw2](5.Docs/2.Images/hw2.jpg)

The keyboard body itself is also a standard custom keyboard layered structure design, including damping cotton, PCBA, positioning plate, switch pad, etc.:

![hw2](5.Docs/2.Images/hw3.jpg)

The keyboard structure design is mainly modified by Xikii based on S98, with a 75-key layout. Students who need other layouts can modify the PCB and firmware to adapt.

> Regarding the structural parts shown in the video, since it's Xikii's solution, I don't feel right to release the source files without permission. Moreover, the original version structure is for CNC machining, which would be quite costly.
>
> So I also asked Xikii to help design a simplified version that can be used for 3D printing, and open source it in the repository.

**About chip selection?**

- The main controller chip chosen for the keyboard is STM32F103CBT6. Actually C8T6 would work too, but considering the future expandability of firmware functions, CBT6 with twice the Flash is more cost-effective. Since I implemented most of the firmware using the HAL library, the main controller can actually be replaced with any chip in the STM32 series, as long as the chip has 2 SPI hardware interfaces for key scanning and RGB light driving respectively, and a full-speed USB interface.
- The STM32F4 for the Dynamic component main controller is because I have a lot of this chip on hand. Theoretically it can be replaced with a more cost-effective F1 series, as long as the chip has an advanced timer for PWM generation, 2 hardware SPI interfaces for encoder and e-ink screen communication, an I2C interface for OLED driving, and a full-speed USB interface.
- For the magnetic encoder chip of the motor, I'm using AS5047P, which is also a very commonly used high-performance magnetic encoder chip, but the cost is slightly higher. I only chose this one because I had it on hand. It can also be modified to other cheaper models such as MT6816, etc., of course, the firmware driver code needs to be modified accordingly.
- The shift register used for key scanning is 74HC165. Domestic chips cost about 0.5 yuan each in retail, and one chip can drive 8 keys. You can modify the number of cascaded register chips according to the number of keys you need. Imported 165, such as TI original, is more expensive than domestic ones and will perform slightly better, but since the key scanning frequency of 4MHz in this project is already more than enough, even domestic 16MHz chips are more than adequate.
- The capacitive touch panel uses a 6-channel capacitive touch key chip XW06A. This has certain requirements for the design of PCB sensing pads. The repository has provided the designed PCB. As for the reading method of this chip, it's actually no different from ordinary keys, so in this scheme it's also scanned and read using 74HC165.
- The motor FOC drive circuit is completely ported from my Ctrl driver, using FD8288Q as the gate driver, without the need for a current sensor.

**About the programming method?**

Use debuggers like JLink, STLink for programming. I have reserved SWD debug ports on both the PCB and the shell. For students without hardware development experience, I will also release a Bootloader later, which can directly upgrade the firmware through the USB port.

**About motor selection?**

I'm using a second-hand 2204 motor, but this model seems to be hard to buy now. You can replace it with a similar sized brushless motor. In terms of parameters, you need a lower KV value, preferably around 200. The motor needs to manually install a radially magnetized permanent magnet on the rotor for encoder positioning. Different motor models require some adjustment of FOC parameters.

![hw2](5.Docs/2.Images/hw4.jpg)

## 3. Software Architecture Description

**About the key mapping method of the keyboard firmware?**

To fully utilize the advantages of the shift register scanning scheme mentioned in the video, the firmware code decouples the PCB Layout wiring and key scanning order, and remaps through software. That is to say, the connection of keys in the PCB can be arbitrary, and after wiring, the mapping method can be specified in `keyMap[KEYMAP_NUM][IO_NUMBER]` in the `hw_keyboard.h` file.

> This is a two-dimensional array, representing `KEYMAP_NUM` layers of key mappings, each layer has `IO_NUMBER` keys (that is, the number of keys on your keyboard); where the 0th layer is special, responsible for mapping the random layout of PCB keys to the standard keyboard layout, and the subsequent 1, 2, 3, 4... layers are all custom, responsible for mapping the standard key layout to any layout.

**An example:**

Consider the key pointed by the arrow in the schematic, this key can be at any position on the PCB, but we can see that it is the 10th from left to right (in the order of 74HC165 connection, i.e. shift scanning order), so its number is 9 (counting from 0).

![hw2](5.Docs/2.Images/hw5.jpg)

If we placed it in the position of **right Alt** on the actual PCB board, then referring to the sequence number 76 of `RIGHT_ALT` in the 1st layer mapping (i.e. standard layout) in the **red box** in the code below, we fill in 9 (blue box) in the 76th variable of the 0th layer mapping.

In this way, fill in all the keys on your PCB into the 0 layer mapping one by one, and you will get a well-mapped standard keyboard. Subsequently, how to map layers 2, 3, 4, 5... can be modified and added at will, and you don't need to use numeric numbers anymore, but can directly use the enumerated key names very conveniently.

> So for people who want to modify the keyboard layout, you only need to add or reduce a few cascaded 74HC165 on the schematic, then wire the PCB arbitrarily, and then delete or add some numbers in the 0 layer mapping in the code (for example, in the example below, my keyboard has 83 keys); the modification of the later layers follows this.

The code uses the `keyboard.Remap` function to map different layers, for example, the sentence `keyboard.Remap(2)` uses the 2nd layer mapping.

![hw2](5.Docs/2.Images/hw6.jpg)

**About the filtering method of the keyboard firmware?**

The firmware uses independent filtering for each key, but it is implemented in a very efficient way (after all, with 1KHz messages, each message period scans the keys at least twice, meaning that **1000\*2\*[number of keys]** times of filtering needs to be performed every second).

The basic principle is very simple. The reason for key bounce is that after pressing, it will repeatedly jump between high and low levels, and this stabilization time is generally tens of us (note that this is the level stabilization time, not the key trigger time, the latter is due to the uncertainty of the key spring contact time, which may last for several ms).

The [qmk_firmware/feature_debounce_type](https://github.com/qmk/qmk_firmware/blob/master/docs/feature_debounce_type.md) document in QMK describes several filtering methods it uses, divided into Eager and Defer, symmetric and asymmetric, etc.

The default is to use **symmetric delay global filtering**, which means applying the same filtering to all keys, waiting until all keys are stable and no longer changing before submitting the scan data.

> In contrast, there's the aggressive filtering method, which submits data as soon as a key change is detected, but doesn't respond to any keys for N milliseconds after that (thus avoiding submitting constantly bouncing keys). This method has low trigger delay but is very sensitive to noise and prone to false triggers.

In HanWen's firmware, I use **symmetric delay independent filtering**, which means performing two detections for each key. If a key change is detected in the first detection, then another detection is performed after N microseconds (this parameter can be configured, just needs to be greater than the typical key bounce time). If the results of the two detections are consistent, then it's determined that the key has been pressed. At this point, we can ensure that the key has changed and won't trigger repeatedly, balancing delay and stability.

This process is efficiently processed through XOR operations. Conveniently, since the key buffer is obtained from shift register scanning, each bit already represents a key, so the filtering efficiency is very high, and the actual effect is quite good.

![hw2](5.Docs/2.Images/hw7.jpg)

**About the HID descriptor of the keyboard firmware?**

You can directly refer to the `usbd_customhid.c` file in the source code. I configured two ReportIDs: ID-0 is for reporting key scan data (N-key rollover), and ID-1 is reserved for future communication with the PC-side key remapping software.

**About RGB control?**

The code uses ws2812b series LEDs with a single bus, where a large number of RGBs can be connected in series with one wire. Moreover, the code implements SPI-DMA simulation timing, achieving an ultra-high refresh rate.

Currently, the code only includes a demo lighting effect (very simple, just cycling through colors). If you want to add additional lighting effects, you can set RGB values through the `keyboard.SetRgbBuffer` function, and then send the data to the LEDs with `SyncLights`:

```cpp
while (true)
{
    /*---- This is a demo RGB effect ----*/
    static uint32_t t = 1;
    static bool fadeDir = true;

    fadeDir ? t++ : t--;
    if (t > 250) fadeDir = false;
    else if (t < 1) fadeDir = true;

    for (uint8_t i = 0; i < HWKeyboard::LED_NUMBER; i++)
        keyboard.SetRgbBuffer(i, HWKeyboard::Color_t{(uint8_t) t, 50, 0});
    /*-----------------------------------*/

    // Send RGB buffers to LEDs
    keyboard.SyncLights();
}
```

## 4. SDK Design & Secondary Development

To be supplemented.

> Thanks to the following projects:
>
> [Lexikos/AutoHotkey_L: AutoHotkey - macro-creation and automation-oriented scripting utility for Windows. (github.com)](https://github.com/Lexikos/AutoHotkey_L)
>
> [olikraus/u8g2: U8glib library for monochrome displays, version 2 (github.com)](https://github.com/olikraus/u8g2)
>
> [simplefoc/Arduino FOC for BLDC  (github.com)](https://github.com/simplefoc/Arduino-FOC)
>
> [zhongyang219/TrafficMonitor: This is a utility that displays the current network speed, CPU and memory usage on the desktop taskbar. It also supports skin customization. (github.com)](https://github.com/zhongyang219/TrafficMonitor)
