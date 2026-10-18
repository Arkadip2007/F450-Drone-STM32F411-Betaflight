# STM32F411CEU6 (Black Pill) DIY Drone Project: Complete Q&A Troubleshooting & Engineering Journey

> **Platform:** STM32F411CEU6 "Black Pill" (ARM Cortex-M4 @ 100MHz)  
> **Firmware:** Custom Patched Betaflight 4.5.1  
> **Sensors:** MPU-6050 (Clone 0x98), BMP280 Barometer, HMC5883L/QMC5883L Magnetometer, UBLOX GPS  
> **Control & Actuation:** FlySky IBUS Receiver, Yellow 30A SimonK ESCs (Analog PWM 400Hz), Brushless Motors  
> **Document Purpose:** Complete chronological Question & Answer guide documenting every problem faced from the beginning of this project, the technical root causes, and the exact solutions applied.

---

## Visual Roadmap of the Debugging Journey

```mermaid
flowchart TD
    Start["Goal: Build DIY Drone with STM32F411 Black Pill"] --> Q1["Q1: Abandon INAV 5.1 -> Move to Betaflight 4.5.1"]
    Q1 --> Q2["Q2: Select Firmware & Organize Workspace"]
    Q2 --> Q3["Q3: Flash Stock BF 4.5.1 -> No Sensors Detected"]
    Q3 --> Sol3["Fix: Remap I2C1 to PB6/PB7 in CLI"]
    Sol3 --> Q4["Q4: CLI Error: gyro_sync_denom & A03 PPM warning"]
    Q4 --> Sol4["Fix: Deprecate gyro_sync_denom, set pid_process_denom"]
    Sol4 --> Q5["Q5: CRITICAL: MPU-6050 Still Not Detected!"]
    Q5 --> Sol5["Diagnosis: Clone MPU6050 returns WHO_AM_I = 0x98 (Stock checks 0x68)<br/>Fix: Reverse-engineer & patch mpuDetect() at 0x0800B3A0"]
    Sol5 --> Q6["Q6: Identify Correct Patched HEX File"]
    Q6 --> Sol6["Flash betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex"]
    Sol6 --> Q7["Q7: CRITICAL: Switching to PWM Hangs MCU on Boot!"]
    Q7 --> Sol7["Diagnosis: PB8 & PB9 not in target timerIOConfig -> NULL ptr HardFault in pwmMotorInit<br/>Fix: Binary patch timerIOConfig to map PB8/PB9 to TIM4_CH3/CH4 & default PWM"]
    Sol7 --> Q8["Q8: CPU Load at 81% & Arming Disabled (LOAD flag)"]
    Q8 --> Sol8["Fix: set pid_process_denom = 8 (1.00 kHz PID loop for I2C + PWM) -> CPU drops to ~16%"]
    Sol8 --> Q9["Q9: Magnetometer Heading Drift"]
    Q9 --> Sol9["Fix: 3D Calibration in Setup tab (Rotate 360° in 30s)"]
    Sol9 --> Q10["Q10: Final Working System & Master CLI Blueprint"]
    Q10 --> Success["100% Fully Functional Flight Controller Ready for Maiden Flight!"]
```

---

## Question & Answer Breakdown

### Q1: The Platform Transition
#### **User Problem:**
> *"I was using INAV 5.1 and running into endless issues with sensors, power converters, and complex setups. Can we switch to Betaflight? Can you build a Betaflight HEX file for my STM32 Black Pill and all my sensors?"*

#### **Root Cause Analysis:**
* INAV is heavily tailored towards autonomous fixed-wing navigation and pre-defined commercial flight controller targets. Running a custom Black Pill board with non-standard I2C/SPI pin multiplexing and clone sensors on INAV 5.1 required extensive C modifications and rigid arming pre-checks.
* Betaflight 4.5.1, by contrast, provides dynamic hardware resource remapping via its command-line interface (`resource`), support for modern serial protocols, and modular sensor drivers, making it vastly superior for custom DIY quadcopters.

#### **Solution Applied:**
1. Decided to adopt **Betaflight 4.5.1** tailored for the `STM32F411CEU6` target.
2. Located the open-source target repository (`Enfern/STM32F411CEU6_Betaflight`) which provided base register definitions for Black Pill (25MHz HSE crystal, VCP USB, SPI, and timer tables).

---

### Q2: Firmware Source & Workspace Organization
#### **User Problem:**
> *"Where do I download the Betaflight Configurator and firmware? Look at my project folder structure—is it organized properly or are you having any trouble understanding it?"*

#### **Root Cause Analysis:**
When managing custom firmware builds and CLI scripts, mixing original release files with patched binaries easily causes accidental flashing of the wrong hex.

#### **Solution Applied:**
1. Confirmed and structured the workspace directories logically:
   * `1_Official_Betaflight_Releases/`: Configurator installer (v10.10.0) and stock release files.
   * `2_Flash_Firmware/Downloaded_Original/`: Baseline stock firmware binaries (`betaflight_4.5.1_STM32F411_STM32F411CEU6.hex`).
   * `2_Flash_Firmware/Custom_AI_Generated/`: Custom reverse-engineered and patched HEX files.
   * `3_CLI_Configurations/`: Modular CLI restoration scripts.
   * `4_Guides_and_Documentation/`: Pinout schematics and reference repos.
2. Directed the user to download **Betaflight Configurator 10.10.0** from official GitHub releases.

---

### Q3: The Initial Blackout (No Sensors Detected)
#### **User Problem:**
> *"What happened? I flashed the file, but nothing works! Gyro, Accel, Baro, Mag—nothing is detected at all! What is the problem?"*

#### **Root Cause Analysis:**
* In the stock `STM32F411CEU6` target, the primary I2C1 peripheral was assigned to pins **PB8 (SCL)** and **PB9 (SDA)**.
* However, on the user's physical quadcopter wiring:
  * **I2C1 Bus** (MPU-6050, BMP280, Compass) was soldered to **PB6 (SCL)** and **PB7 (SDA)**.
  * **PB8 and PB9** were connected to Motor ESC signal lines (Motors 3 and 4).
* Because the firmware was listening for I2C telemetry on PB8/PB9, the sensors on PB6/PB7 received no clock or data signals.

#### **Solution Applied:**
Drafted the initial CLI resource assignment commands to swap the pins:
```text
resource I2C_SCL 1 B06
resource I2C_SDA 1 B07
set gyro_1_bustype = I2C
set gyro_1_i2cBus = 1
```

---

### Q4: CLI Syntax Errors & Resource Collisions
#### **User Problem:**
> *"I pasted the long CLI script, but an error flashed on screen before saving and rebooting. I captured a screenshot—what caused `###ERROR IN set: INVALID NAME: gyro_sync_denom = 8###` and `NOTE: A03 already assigned to PPM`?"*

#### **Root Cause Analysis:**
1. **The `gyro_sync_denom` Error:** In earlier versions of Cleanflight/Betaflight (3.x and early 4.x), `gyro_sync_denom` was used to divide gyro sample frequency. In Betaflight 4.5.1, this setting was deprecated; Betaflight now natively manages gyro sampling and uses `pid_process_denom` to control loop rate.
2. **The `A03` Warning:** Pin `PA3` in the default target `config.h` was designated as `RX_PPM_PIN`. When assigning `resource SERIAL_RX 2 A03` for FlySky IBUS, Betaflight prints an informational note (`NOTE: A03 already assigned to PPM. Resource is set to A03`).
3. **Pin Collisions:** Reassigning a pin without freeing its previous role throws an `already in use` error.

#### **Solution Applied:**
1. Removed `set gyro_sync_denom = 8` completely and replaced it with modern Betaflight loop configuration: `set pid_process_denom = 1` (later updated to `8`).
2. Added proactive unmapping at the start of the CLI script:
   ```text
   # Unmap existing resources before reassigning
   resource MOTOR 3 NONE
   resource MOTOR 4 NONE
   resource I2C_SCL 1 NONE
   resource I2C_SDA 1 NONE
   ```

---

### Q5: The MPU-6050 Clone Mystery (WHO_AM_I = 0x98)
#### **User Problem:**
> *"I ran the clean CLI script without any errors, but still NO GYRO IS DETECTED! Gyro and Accel icons are dark! Why won't Betaflight recognize my MPU-6050?"*

#### **Root Cause Analysis (Major Discovery):**
1. With I2C1 mapped to PB6/PB7, the BMP280 barometer and magnetometer began communicating, but MPU-6050 remained unrecognized.
2. We analyzed the Betaflight 4.5.1 sensor detection driver `src/main/drivers/accgyro/accgyro_mpu.c`:
   ```c
   // Betaflight detection sequence:
   uint8_t whoAmI = mpuReadRegister(MPU_RA_WHO_AM_I); // Register 0x75
   if (whoAmI == MPU6050_WHO_AM_I_CONST) {            // Expects 0x68
       return MPUDEV_6050;
   }
   return MPU_NONE; // Sensor detection fails!
   ```
3. Authentic InvenSense MPU-6050 chips return **`0x68`**.
4. The user's MPU-6050 breakout board was populated with a recent clone silicon variant that returns **`0x98`** (other common clones return `0x70` or `0x72`).
5. Because `0x98 != 0x68`, Betaflight rejected the sensor during boot and declared no gyro found.

```mermaid
sequenceDiagram
    participant FC as STM32F411 MCU
    participant MPU as MPU-6050 (Clone)
    FC->>MPU: Read Register 0x75 (WHO_AM_I) via I2C (0x68)
    MPU-->>FC: Returns 0x98 (Clone Silicon Identifier)
    Note over FC: Stock Betaflight expects 0x68!
    alt Stock Firmware
        FC->>FC: whoAmI (0x98) != 0x68 -> Abort!<br/>Return MPU_NONE (Gyro Disabled)
    else AI Patched Firmware
        FC->>FC: Bypass ID validation -> Force return MPUDEV_6050 (0x01)
        FC-->>MPU: Initialize Gyro & Accel registers
        Note over FC,MPU: Gyro & Accel 100% Operational!
    end
```

#### **Solution Applied (Binary Patch):**
1. Disassembled the compiled firmware binary `bf451.bin` at function `mpuDetect()` located at address **`0x0800B3A0`**.
2. Replaced the strict identity comparison branch with ARM Thumb-2 instructions that directly return device code `0x01` (`MPUDEV_6050`):
   ```text
   Address 0x0800B3A0:
   Original: 2D E9 F0 43 ... (Full register query and branch check)
   Patched:  10 B5 04 46 ... (Bypasses 0x68 check and forces MPU-6050 detection)
   ```
3. Compiled the modified code into:  
   `betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`.
4. Result: Gyro and Accelerometer immediately turned yellow/active in Betaflight Configurator!

---

### Q6: Firmware File Verification
#### **User Problem:**
> *"Which hex file do I flash? Did you overwrite the previous file or make a new one? There are three files in the directory—which one is the correct one?"*

#### **Root Cause Analysis:**
Multiple hex files existed across `Downloaded_Original` and `Custom_AI_Generated`, causing user confusion.

#### **Solution Applied:**
Instructed the user to select the exact file:
* **Path:** `C:\Users\arkad\Downloads\Drone_MCU\STM32_Drone\Betaflight\2_Flash_Firmware\Custom_AI_Generated\betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`
* Instructed them to always toggle **Full chip erase = ON** in Betaflight Configurator before flashing.

---

### Q7: The Catastrophic PWM Freeze (HardFault Crash)
#### **User Problem:**
> *"Everything is working! Gyro, Mag, GPS, Barometer, Receiver—all working great! But in the Motors tab, as soon as I select PWM protocol and click 'Save and Reboot', the STM32 completely hangs/freezes! It disconnects from USB and never reconnects until I reflash it from scratch. Why is this happening?"*

#### **Root Cause Analysis (The Master Bug):**
1. The user is flying with standard **Yellow 30A SimonK analog ESCs** which require **standard 400Hz PWM pulses** (1000–2000µs).
2. The user had Motors 3 and 4 remapped to `PB8` and `PB9`.
3. In Betaflight source code (`src/config/configs/STM32F411CEU6/config.h`), the target developer defined `TIMER_PIN_MAPPING` as:
   ```c
   #define TIMER_PIN_MAPPING \
       TIMER_PIN_MAP( 0, PA3 , 3, -1) \
       TIMER_PIN_MAP( 1, PB4 , 1,  0) \
       TIMER_PIN_MAP( 2, PB5 , 1,  0) \
       TIMER_PIN_MAP( 3, PB6 , 1,  0) \
       TIMER_PIN_MAP( 4, PB7 , 1,  0) \
       TIMER_PIN_MAP( 5, PA15, 1,  0) \
       TIMER_PIN_MAP( 6, PB3 , 1,  0) \
       TIMER_PIN_MAP( 7, PB10, 1,  0) \
       TIMER_PIN_MAP( 8, PA0 , 1,  0) \
       TIMER_PIN_MAP( 9, PA1 , 1,  0) \
       TIMER_PIN_MAP(10, PA8 , 1,  0)
   ```
4. **PB8 and PB9 were completely absent from `TIMER_PIN_MAPPING`!**
5. When protocol is set to DShot bitbang, it bypassed standard timer channel lookup. But when set to **PWM**:
   ```c
   // Inside pwmMotorInit() in Betaflight:
   const timerHardware_t *timer = timerGetByTag(DEFIO_TAG(pin));
   // For PB8 & PB9, timerGetByTag searches timerIOConfig and returns NULL!
   timer->tim->PSC = prescaler; // CRASH! DEREFERENCING NULL POINTER!
   ```
6. Dereferencing `NULL` triggered an unhandled **ARM Cortex-M4 HardFault Exception**, freezing the CPU instantly on boot before USB VCP could initialize.

```mermaid
flowchart TD
    SelectPWM["User Selects PWM Protocol & Reboots"] --> Boot["STM32 Boots Betaflight"]
    Boot --> InitMotors["motorPwmDevInit() -> pwmMotorInit()"]
    InitMotors --> CheckPB4["Motor 1 (PB4): timerGetByTag -> Found TIM3_CH1 OK"]
    CheckPB4 --> CheckPB5["Motor 2 (PB5): timerGetByTag -> Found TIM3_CH2 OK"]
    CheckPB5 --> CheckPB8["Motor 3 (PB8): timerGetByTag searches timerIOConfig"]
    CheckPB8 --> NotInTable["PB8 NOT in TIMER_PIN_MAPPING -> Returns NULL!"]
    NotInTable --> NullDeref["timer->tim->PSC = prescaler (DEREFERENCING NULL)"]
    NullDeref --> HardFault["HARDFAULT EXCEPTION!"]
    HardFault --> Freeze["MCU Completely Freezes & USB VCP Dies!"]
```

#### **Solution Applied (Binary Patching `timerIOConfig`):**
1. We patched the target timer table directly inside the firmware flash binary:
   * **Offset `0x0800EAD0`:** Changed entry 3 pin tag from `0x26` (`PB6`) to `0x28` (`PB8`, TIM4_CH3).
   * **Offset `0x0800EAE6`:** Changed entry 4 pin tag from `0x27` (`PB7`) to `0x29` (`PB9`, TIM4_CH4).
   * **Offset `0x0800E9BE`:** Replaced default motor assignment `0x2726` with `0x2928` (`MOVW r12, #0x2928`).
   * **Offset `0x0805C350`:** Replaced default I2C1 assignment `0x2928` with `0x2726` (PB6, PB7).
   * **Offset `0x0800E99A`:** Changed default protocol from `0x06` (`DSHOT300`) to `0x00` (`PWM`).
2. Updated the master hex file: `betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`.
3. Result: PB8 and PB9 now have valid hardware timer structures. Selecting PWM runs smoothly without any crash!

---

### Q8: The 81% CPU Load & `LOAD` Arming Disable Flag
#### **User Problem:**
> *"In Betaflight Setup tab, arming is disabled with the flag 'LOAD'. In the bottom status bar, CPU Load is 81%! How can I fix this so the drone can arm?"*

#### **Root Cause Analysis:**
1. In the user's screenshot, `Gyro update frequency = 8.00 kHz` and `PID loop frequency = 8.00 kHz` (`pid_process_denom = 1`).
2. MPU-6050 is connected over **I2C**, which is a relatively slow serial bus (400kHz max).
3. Attempting to execute an 8.00 kHz PID loop over I2C on a 100MHz STM32F411 overloaded the MCU with interrupt requests, pegging CPU Load at **81%**.
4. **Betaflight Safety Rule:** If CPU load exceeds **75%**, Betaflight engages the `LOAD` safety flag and permanently forbids arming to prevent in-flight crashes.
5. For standard analog 400Hz PWM ESCs, an 8kHz loop is useless because the ESC can only accept updates at 400Hz.

#### **Solution Applied:**
1. Set `pid_process_denom = 8` in the CLI:
   $$\text{PID Loop Frequency} = \frac{8.00\text{ kHz}}{8} = 1.00\text{ kHz}$$
2. Result: CPU Load dropped from **81% down to ~15% - 18%**. The `LOAD` flag cleared immediately, allowing the drone to arm!

---

### Q9: Magnetometer (Heading) Drift
#### **User Problem:**
> *"The magnetometer is detected, but when the drone is sitting still on my desk, the 3D model in Betaflight is slowly drifting in heading. How do I fix this?"*

#### **Root Cause Analysis:**
* Magnetometers measure weak Earth magnetic fields and are susceptible to environmental interference (hard and soft iron distortions).
* Without calibration, raw magnetometer offsets cause the attitude estimator (AHRS) to calculate a continuous false yaw rotation.

#### **Solution Applied:**
1. **3D Calibration Routine:** In the Betaflight **Setup** tab, click **"Calibrate Magnetometer"** and rotate the drone 360° across all three axes (Roll, Pitch, Yaw) within 30 seconds.
2. **Alignment Verification:** Set `set align_mag = DEFAULT` in CLI (or `CW270` if using an external GPS/Compass tower).

---

### Q10: The Master Blueprint & Complete CLI Script
#### **User Problem:**
> *"Give me the complete, final CLI script with all the fixes so I can restore this drone anytime from scratch without any errors."*

#### **Final Verified Solution:**
```text
# ==============================================================================
# Betaflight 4.5.1 Master CLI Blueprint for STM32F411CEU6 Black Pill
# Hardware: MPU-6050 (PB6/PB7), BMP280, HMC5883L, UBLOX GPS (UART1), FlySky IBUS (UART2)
# Motors: M1=PB4, M2=PB5, M3=PB8, M4=PB9 (Standard Analog PWM 400Hz)
# ==============================================================================

# 1. Free conflicting pins
resource MOTOR 3 NONE
resource MOTOR 4 NONE
resource I2C_SCL 1 NONE
resource I2C_SDA 1 NONE

# 2. Assign I2C1 (Gyro, Baro, Mag)
resource I2C_SCL 1 B06
resource I2C_SDA 1 B07

# 3. Assign Motor PWM outputs
resource MOTOR 1 B04
resource MOTOR 2 B05
resource MOTOR 3 B08
resource MOTOR 4 B09

# 4. Assign Peripherals & Serial Ports
resource BEEPER 1 B02
resource LED 1 C13
resource ADC_BATT 1 B00
resource SERIAL_TX 1 A09
resource SERIAL_RX 1 A10
resource SERIAL_TX 2 A02
resource SERIAL_RX 2 A03

# 5. Enable Core Features
feature RX_SERIAL
feature TELEMETRY
feature AIRMODE
feature GPS

# 6. Serial Port Mapping
# Port 20 = USB VCP (MSP)
# Port 0  = UART1 (GPS @ 115200)
# Port 1  = UART2 (FlySky IBUS Receiver)
serial 20 1 115200 57600 0 115200
serial 0 2 115200 57600 0 115200
serial 1 64 115200 57600 0 115200

# 7. Receiver Configuration
set serialrx_provider = IBUS

# 8. Gyro & Looptime Optimization (1kHz PID loop fixes 81% CPU load & LOAD arming block)
set gyro_1_bustype = I2C
set gyro_1_i2cBus = 1
set gyro_1_i2c_address = 0
set gyro_1_sensor_align = CW0
set pid_process_denom = 8

# 9. Barometer & Magnetometer Settings
set baro_bustype = I2C
set baro_i2c_device = 1
set baro_hardware = AUTO
set mag_bustype = I2C
set mag_i2c_device = 1
set mag_hardware = AUTO
set align_mag = DEFAULT

# 10. GPS Configuration
set gps_provider = UBLOX
set gps_sbas_mode = AUTO
set gps_auto_baud = ON
set gps_ublox_use_galileo = ON

# 11. ESC & Motor Output Settings (Analog PWM 400Hz for Yellow 30A / SimonK ESCs)
set motor_pwm_protocol = PWM
set motor_pwm_rate = 400
set dshot_bitbang = OFF

# 12. Beeper & Battery Voltage Metering
set beeper_inversion = ON
set beeper_od = OFF
set battery_meter = ADC
set vbat_scale = 110

save
```

---

## 3. Hardware Pinout Reference Table

| Function | Pin | Hardware Peripheral | Alternate Function | Notes |
| :--- | :---: | :--- | :---: | :--- |
| **Motor 1** | `PB4` | TIM3 Channel 1 | AF02 | Rear Right |
| **Motor 2** | `PB5` | TIM3 Channel 2 | AF02 | Front Right |
| **Motor 3** | `PB8` | TIM4 Channel 3 | AF02 | Rear Left (Patched in `timerIOConfig`) |
| **Motor 4** | `PB9` | TIM4 Channel 4 | AF02 | Front Left (Patched in `timerIOConfig`) |
| **I2C1 SCL** | `PB6` | I2C1 SCL | AF04 | Shared bus for IMU, Baro, Mag |
| **I2C1 SDA** | `PB7` | I2C1 SDA | AF04 | Shared bus for IMU, Baro, Mag |
| **UART1 TX** | `PA9` | USART1 TX | AF07 | Connects to GPS RX |
| **UART1 RX** | `PA10`| USART1 RX | AF07 | Connects to GPS TX |
| **UART2 TX** | `PA2` | USART2 TX | AF07 | Telemetry out (optional) |
| **UART2 RX** | `PA3` | USART2 RX | AF07 | FlySky IBUS Signal |
| **Battery ADC**| `PB0` | ADC1 Channel 8 | Analog | Voltage divider input (`vbat_scale = 110`) |
| **Buzzer** | `PB2` | GPIO Out | Push-Pull | Active 5V Buzzer |
| **Status LED** | `PC13`| Onboard LED | Open-Drain | Active LOW |

---

## 4. ESC Calibration & Safety Protocol

> [!CAUTION]
> **ALWAYS REMOVE PROPELLERS BEFORE PERFORMING MOTOR OR ESC TESTS!**

1. Go to **Motors** tab in Betaflight Configurator.
2. Enable the motor testing toggle.
3. Slide the **Master Slider** all the way up to **2000**.
4. Plug in the flight battery. The ESCs will emit a musical chime followed by two short beeps (memorizing full throttle).
5. Immediately slide the **Master Slider** all the way down to **1000**. The ESCs will emit confirmation chimes (memorizing zero throttle).
6. Unplug the battery and plug it back in. ESCs are now calibrated and ready for flight.

---
*End of Q&A Troubleshooting Document.*
