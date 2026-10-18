# 📸 Visual Documentation & Hardware Gallery

This directory hosts high-resolution screenshots, physical hardware build captures, schematics, and diagrams from actual bench-testing and flight-tuning sessions of the **F450 STM32F411 Black Pill Betaflight Drone**.

---

## 🖼️ Gallery of Images

### A. Configurator & Firmware Telemetry

#### 1. [Betaflight Setup & Sensor Health Overview](betaflight-setup-model-sensors.png)
- **File:** `betaflight-setup-model-sensors.png`
- **Configurator Tab:** `Setup`
- **Key Details Shown:**
  - Real-time 3D Quad-X model rendering reacting to gyro/accel movement
  - Attitude readouts: Heading, Pitch (-0.1°), Roll (0.5°)
  - Active sensor status indicators on top header:
    - **GYRO**: Enabled & Healthy (Green)
    - **ACCEL**: Enabled & Healthy (Green)
    - **MAG**: Enabled & Healthy (Green)
    - **BARO**: Enabled & Healthy (Green)
    - **GPS**: Enabled & Healthy (Green)
  - System Arming Disable flags showing `CLI` and `MSP` (Normal bench state when connected via USB)
  - Voltage telemetry (12.3V 3S LiPo readout)

#### 2. [Betaflight Configuration & Sensor Alignment](betaflight-configuration-tab.png)
- **File:** `betaflight-configuration-tab.png`
- **Configurator Tab:** `Configuration`
- **Key Details Shown:**
  - **Mixer**: Quad X motor configuration
  - **ESC/Motor Features**: Protocol set to standard `PWM` (400Hz for SimonK 30A ESCs)
  - **Board and Sensor Alignment**:
    - Gyro/Accelerometer: `CW 0°` (Default orientation)
    - Magnetometer: `CW 180° Flip` (Required for GY-271 sensor orientation)
  - **System Configuration**:
    - PID loop frequency: `1.00 kHz` (`set pid_process_denom = 8`) to maintain ~15% CPU load on 96MHz STM32F411 without bus contention
  - **Receiver Protocol**: Serial-based receiver set to `IBUS`

---

### B. Physical Drone Hardware & Bench Testing

#### 3. [Complete Drone Bench Setup & Configurator Link](drone-bench-setup-laptop.jpg)
- **File:** `drone-bench-setup-laptop.jpg`
- **Description:** Complete assembled F450 quadcopter connected to the laptop workstation via USB for live sensor calibration, motor direction testing, and receiver telemetry verification.

#### 4. [Custom DIY STM32F411 Flight Controller Board (Top View)](stm32f411-flight-controller-board-top.jpg)
- **File:** `stm32f411-flight-controller-board-top.jpg`
- **Description:** Close-up of the custom-soldered prototyping flight controller board featuring:
  - STM32F411CEU6 Black Pill (center)
  - MPU-6050 6-DOF IMU module
  - BMP280 barometric altimeter module
  - GY-271 magnetometer compass module
  - u-blox NEO-6M GPS receiver module

#### 5. [Flight Controller Installed with Active Power & Status LEDs](stm32f411-fc-installed-wiring.jpg)
- **File:** `stm32f411-fc-installed-wiring.jpg`
- **Description:** The flight controller board mounted on the central F450 plate, powered up via ESC 5V BEC with status LEDs illuminated, confirming clean power delivery and active firmware execution.

#### 6. [F450 Quadcopter Airframe Assembly](f450-frame-assembly.jpg)
- **File:** `f450-frame-assembly.jpg`
- **Description:** Mechanical assembly of the 450mm wheelbase fiberglass central plates and reinforced polyamide arms prior to electronic component installation.

#### 7. [All-Up-Weight (AUW) Measurement on Digital Scale](f450-drone-weight-scale.jpg)
- **File:** `f450-drone-weight-scale.jpg`
- **Description:** Complete quadcopter on precision digital scale validating actual All-Up-Weight (AUW) with battery and propellers (~1.1 kg), confirming the calculated ~2.5:1 thrust-to-weight ratio.

#### 8. [FlySky FS-i6 Radio Transmitter](flysky-fs-i6-transmitter.jpg)
- **File:** `flysky-fs-i6-transmitter.jpg`
- **Description:** FlySky FS-i6 2.4GHz AFHDS 2A radio transmitter configured for Mode 2 (Throttle on left) with 14-channel i-BUS serial output enabled.

#### 9. [FlySky Transmitter Internal Power Supply Upgrade](flysky-transmitter-power-supply-mod.jpg)
- **File:** `flysky-transmitter-power-supply-mod.jpg`
- **Description:** Internal hardware modification replacing standard disposable AA batteries with a rechargeable **Lithium-Ion battery power supply**, providing extended operational battery life during extensive flight-tuning sessions.

---

### C. Pinout & Reference Diagrams

#### 10. [STM32F411CEU6 Black Pill Pinout Diagram](stm32f411-blackpill-pinout.png)
- **File:** `stm32f411-blackpill-pinout.png`
- **Purpose:** Complete hardware reference diagram detailing GPIO pins, timer channels (TIM1, TIM2, TIM3, TIM4), alternate functions, ADC channels, and 5V tolerance.

---

## 🔗 Cross-References
- [Main README](../../README.md)
- [Wiring Guide](../wiring-guide.md)
- [Master Hardware Guide](../master-hardware-guide.md)
- [Engineering Q&A Troubleshooting Journey](../engineering-qa-troubleshooting.md)
