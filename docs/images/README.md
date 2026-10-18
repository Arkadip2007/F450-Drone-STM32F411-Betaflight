# 📸 Visual Documentation & Configurator Screenshots

This directory hosts high-resolution screenshots, diagrams, and hardware reference captures from actual bench-testing and flight-tuning sessions of the **F450 STM32F411 Black Pill Betaflight Drone**.

---

## 🖼️ Gallery of Images

### 1. [Betaflight Setup & Sensor Health Overview](betaflight-setup-model-sensors.png)
- **File:** `betaflight-setup-model-sensors.png`
- **Configurator Tab:** `Setup`
- **Key Details Shown:**
  - Real-time 3D Quad-X model rendering reacting to gyro/accel movement
  - Attitude readouts: Heading, Pitch (-0.1°), Roll (0.5°)
  - Active sensor status indicators on the top header:
    - **GYRO**: Enabled & Healthy (Green)
    - **ACCEL**: Enabled & Healthy (Green)
    - **MAG**: Enabled & Healthy (Green)
    - **BARO**: Enabled & Healthy (Green)
    - **GPS**: Enabled & Healthy (Green)
  - System Arming Disable flags showing `CLI` and `MSP` (Normal bench state when connected via USB)
  - Voltage telemetry (12.3V 3S LiPo readout)

---

### 2. [Betaflight Configuration & Sensor Alignment](betaflight-configuration-tab.png)
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

### 3. [STM32F411CEU6 Black Pill Pinout Diagram](stm32f411-blackpill-pinout.png)
- **File:** `stm32f411-blackpill-pinout.png`
- **Purpose:** Complete hardware reference diagram detailing GPIO pins, timer channels (TIM1, TIM2, TIM3, TIM4), alternate functions, ADC channels, and 5V tolerance.

---

## 🔗 Cross-References
- [Main README](../../README.md)
- [Wiring Guide](../wiring-guide.md)
- [Instructions & Setup Guide](../instructions.md)
- [Engineering Q&A Troubleshooting Journey](../engineering-qa-troubleshooting.md)
