# DIY Emo Desk Pet Robot Repository

A detailed documentation of the custom DIY Emo Robot project built with an **ESP32-S3** microcontroller board, a split-voltage power delivery grid, and specialized digital audio/motion tracking modules. 

This repository covers the complete physical hardware layout, pin assignments, structural routing notes, and specific soldering fixes used to create a stable three-point castor wheel layout.

---

## 🔋 1. System Components & Power Grid Architecture

The robot utilizes a dual-rail power grid configured via independent buck converters to keep high-current motor feedback completely isolated from sensitive 3.3V logic signals.

### Core Component Inventory
1. **Microcontroller:** ESP32-S3 Development Board (Uses absolute numeric pin numbers in code).
2. **Main Battery Pack:** ProRange 7.4V 5000mAh Lithium-Polymer (LiPo) 2S Pack.
3. **Primary Isolation Switch:** 12V 20A Heavy Duty Toggle Switch.
4. **Safety System:** Inline 3A Fast-Blow Fuse protecting the primary battery lead.
5. **Drivetrain Buck Converter:** MP1584 Buck Converter dialed to a **6.2V Motor Rail**.
6. **Electronics Buck Converter:** TPS5430D Buck Converter dialed to a **5.2V Logic Rail**.
7. **Motor Driver:** TB6612FNG Dual H-Bridge Motor Driver (Features 3 on-board physical GND pads).
8. **Actuators:** 2x N20 6V 100RPM Micro Gear Motors equipped with integrated Hall-Effect Dual Channel Wheel Encoders.
9. **Chassis Support:** 1x Small Statically-Stable Frictionless Front Castor Wheel.
10. **HMI Display Panel:** 1.3-inch TFT ST7789 240x240 SPI Panel (No-CS Version).
11. **Audio Output:** MAX98357A I2S Mono Class-D Amplifier Breakout paired with a 3W 4-Ohm Speaker.
12. **Audio Input:** INMP441 I2S Omnidirectional Digital Microphone Breakout.
13. **Motion / Orientation Tracking:** MPU6050 6-Axis I2C Inertial Measurement Unit (IMU).
14. **Environmental Detection:** HC-SR04 Ultrasonic Distance Sensor.

### The Split-Voltage System Layout
* **6.2V Drivetrain Rail (via MP1584):** Directly supplies the `VM` high-current pin on the TB6612FNG driver to spin the N20 gear motors.
* **5.2V Electronics Rail (via TPS5430D):** Directly feeds the `VCC` logic gate pin on the TB6612FNG, the `VCC` pin on the HC-SR04 sensor, the `VDD` pin on the MAX98357A Amplifier, and runs straight to the **5V / VIN pin** on the ESP32-S3 to power up the brain wirelessly.

---

## 🔌 2. Complete Project Master Wiring Matrix

All connectivity mappings have been specifically designed using customized open pin designations, bypassing physical hardware locks (**Pin 4 Solder Lock**) and existing trace configurations (**Pin 5 Occupied**).

| Component Module | Module Component Pin | Target Connection Source / ESP32-S3 Pin | Core Purpose & Assembly Logic |
| :--- | :--- | :--- | :--- |
| **TB6612FNG Driver** | **VM** | **6.2V Power Rail** (MP1584 Output) | High-current raw motor switching supply. |
| | **VCC** | **5.2V Power Rail** (TPS5430D Output) | Powers the driver's internal logic gates. |
| | **GND** | **ESP32-S3 GND / Common Rail** | Tied together to build the common ground baseline. |
| | **STBY** | **Pin 8** | Standby control (Pulled `HIGH` in code to enable driver). |
| | **PWMA** | **Pin 9** | Left motor speed control (PWM frequency channel). |
| | **PWMB** | **Pin 10** | Right motor speed control (PWM frequency channel). |
| | **AIN1** | **Pin 11** | Left motor rotation direction input line 1. |
| | **AIN2** | **Pin 12** | **Hard Solder-Locked Link** to control direction. |
| | **BIN1** | **Pin 13** | Right motor rotation direction input line 1. |
| | **BIN2** | **Pin 15** | Right motor rotation direction input line 2 (**Skipped Pin 14**). |
| **Left N20 Motor** | **M1 (Motor +)** | **AO1** (TB6612FNG Driver Output) | Left side main drive positive phase channel. |
| | **M2 (Motor -)** | **AO2** (TB6612FNG Driver Output) | Left side main drive negative phase channel. |
| **Right N20 Motor** | **M1 (Motor +)** | **BO1** (TB6612FNG Driver Output) | Right side main drive positive phase channel. |
| | **M2 (Motor -)** | **BO2** (TB6612FNG Driver Output) | Right side main drive negative phase channel. |
| **Left N20 Encoder** | **VCC** | **3.3V Pin** (ESP32-S3 Output) | Safe low-noise encoder logic supply line. |
| | **GND** | **Common GND Rail** | Closed-loop circuit ground return path. |
| | **OUT A** | **Pin 16** | Left wheel encoder phase A interrupt interrupt hook. |
| | **OUT B** | **Pin 17** | Left wheel encoder phase B direction monitoring line. |
| **Right N20 Encoder**| **VCC** | **3.3V Pin** (ESP32-S3 Output) | Safe low-noise encoder logic supply line. |
| | **GND** | **Common GND Rail** | Closed-loop circuit ground return path. |
| | **OUT A** | **Pin 19** | Right wheel encoder phase A interrupt interrupt hook. |
| | **OUT B** | **Pin 20** | Right wheel encoder phase B direction monitoring line. |
| **HC-SR04 Sensor** | **VCC** | **5.2V Power Rail** (TPS5430D Output) | Powering internal acoustic wave generator circuitry. |
| | **GND** | **Common GND Rail** | Sensor circuit ground alignment point. |
| | **Trig** | **Pin 18** | **Moved from blocked Pin 4!** Trigger signal link. |
| | **Echo** | **Pin 7** | **Moved from occupied Pin 5!** *(Needs 3.3V Divider)*. |
| **INMP441 Microphone**| **VDD** | **3.3V Pin** (ESP32-S3 Output) | Dedicated ultra-clean 3.3V regulator supply. |
| | **GND** | **Common GND Rail** | Microphone logic ground return tracking. |
| | **L/R** | **Common GND Rail** | Hardwired directly to map to Left audio signal path. |
| | **WS** | **Pin 41** | I2S Word Select line (Audio clock timeline timing). |
| | **SCK** | **Pin 42** | I2S Serial Clock line for synchronous data transmission. |
| | **SD** | **Pin 2** | I2S Serial Data Output bitstream link. |
| **I2S Audio Amp** | **VDD** | **5.2V Power Rail** (TPS5430D Output) | Direct high-efficiency audio power input feed. |
| | **GND** | **Common GND Rail** | Audio circuit amplifier reference ground. |
| | **LRCLK** | **Pin 45** | I2S Word Select line (Audio frame boundary clock). |
| | **BCLK** | **Pin 47** | I2S Bit Clock line for bitstream transmission sync. |
| | **DIN** | **Pin 21** | Digital audio serial input data stream line. |
| | **GAIN** | *Leave Floating / Disconnected* | Automatically keeps operational thresholds at 9dB gain. |
| | **SD** (Shutdown) | *Leave Floating / Disconnected* | Automatically mixes Left + Right audio paths into Mono. |
| **ST7789 TFT Face** | **GND** | **Common GND Rail** | Display tracking ground layout return. |
| | **VCC** | **3.3V Pin** (ESP32-S3 Output) | Clean internal controller and logic level power. |
| | **SCL** | **Pin 36** | SPI Hardware Clock communications line tracking. |
| | **SDA** | **Pin 35** | SPI MOSI Serial Data communications line tracking. |
| | **RES** | **Pin 37** | Physical display hardware reset command tracking. |
| | **DC** | **Pin 38** | Data/Command selector line for fast array processing. |
| | **BLK** | **3.3V Pin** | Tied with VCC to 3.3V to keep backlight safe. |
| **MPU6050 IMU** | **VCC** | **3.3V Pin** (ESP32-S3 Output) | Safe digital processor operational voltage input. |
| | **GND** | **Common GND Rail** | Inertial motion logic ground path track. |
| | **SCL** | **Pin 9** | I2C Hardware Serial Clock line for timing. |
| | **SDA** | **Pin 8** | I2C Hardware Serial Data line for vector packet streaming. |
| | **XDA / XCL** | *Leave Disconnected* | Auxiliary sensor routing (Unused for this robot). |
| | **ADO / INT** | *Leave Disconnected* | Forces default address registry configuration to `0x68`. |

---

## 🛠️ 3. Critical Solder Notes & Hardware Modifications

### ⚠️ Single GND Hardware Solution
The specific variant of the ESP32-S3 breakout board used features **only one single physical ground pin labeled GND**. Because the TB6612FNG dual motor driver, digital I2S sensors, SPI screen, and dual logic buck networks all demand an absolute common ground baseline to interface data loops safely, the following physical modification was executed:
* **The Solder Fix:** Jumper wires running from the TB6612FNG motor driver's physical ground ports were **soldered directly together into a unified joint with the ESP32-S3's single GND pin**.
* This joins the return baseline for the logic gates and power switching circuits, guaranteeing the microcontroller can pull logic states high/low without floating voltages disrupting communications.

### Space-Saving Internal Stacking
Because this robot uses hand-soldered **flexible jumper wires** inside a compact 3D-printed case, layout design must optimize clearance constraints to ensure housing halves close seamlessly without stress:
1. **Vertical Triple-Tier Alignment:** The heavy ProRange battery and twin N20 motors line the basement floor for physical center-of-gravity stabilization. Microcontroller, drivers, and step-downs sit centered on a mid-level shelf, while the ST7789 display sits dead-flush facing the outward facial bezel aperture.
2. **90-Degree Parallel Solder Leads:** All jumper connections routed to the back layout of the TFT display and ESP32-S3 are soldered down completely **parallel to the PCB surfaces** at a flat 90-degree fold rather than standing straight up.
3. **Flat Ribbon Gathering:** Multi-wire data buses (like the SPI block and I2S lines) are preserved in their native, flat flat-ribbon configurations to run cleanly along the internal side walls of the chassis frame.

---

## 🔌 4. Type-C Zero-Wire Balance Charging Protocol

To protect the longevity of the dual-cell battery array without adding structural wire weight or bloating your budget:
* **The Selected Solution:** A pocket-sized **Type-C 2S 8.4V LiPo Charger Board** is utilized, featuring an on-board input Type-C socket on one end and a native **3-pin XH2.54 balance terminal block** directly surface-mounted onto the green PCB.
* **The Routine:** No extra wire bundles are added to the pack. Flip the robot's main 12V switch to **OFF**. Slide the battery's factory-stock white **3-pin JST-XH balance plug** directly into the board's slot, insert a smartphone Type-C wire, and let the integrated IC balance both internal cells safely to exactly 4.2V each.
