# STM32F411 Black Pill Betaflight Complete Wiring Guide

This guide describes how to connect all sensors, receivers, and ESCs to the STM32F411CEU6 Black Pill flight controller running Betaflight.

---

## 1. I2C Sensor Bus (PB6 & PB7)

All three I2C modules connect in parallel to `PB6` (SCL) and `PB7` (SDA):

| Sensor | Pin on Sensor | STM32F411 Black Pill Pin | Electrical Notes |
| :--- | :--- | :--- | :--- |
| **MPU-6050** | SCL | **PB6** | Shared hardware I2C1 |
| **MPU-6050** | SDA | **PB7** | Shared hardware I2C1 |
| **BMP280** | SCL | **PB6** | Parallel connection |
| **BMP280** | SDA | **PB7** | Parallel connection (I2C addr: `0x76`) |
| **QMC5883L / HMC5883L** | SCL | **PB6** | Parallel connection |
| **QMC5883L / HMC5883L** | SDA | **PB7** | Parallel connection (I2C addr: `0x0D` / `0x1E`) |
| All Sensors | VCC | **3.3V** | Powered from Black Pill 3.3V output |
| All Sensors | GND | **GND** | Common ground |

---

## 2. Quad-X Motor / ESC PWM Connections

The 4 ESC signal wires connect directly to the hardware timer PWM outputs:

| Motor Number | Quad-X Position | Rotation Direction | STM32 Pin | Hardware Timer Channel |
| :--- | :--- | :--- | :--- | :--- |
| **Motor 1** | Rear-Right | Counter-Clockwise (CCW) | **PB4** | TIM3_CH1 |
| **Motor 2** | Front-Right | Clockwise (CW) | **PB5** | TIM3_CH2 |
| **Motor 3** | Rear-Left | Clockwise (CW) | **PB8** | TIM4_CH3 |
| **Motor 4** | Front-Left | Counter-Clockwise (CCW) | **PB9** | TIM4_CH4 |

---

## 3. Radio Receiver (FlySky i-BUS)

The FlySky serial receiver connects to hardware UART2:

| Receiver Wire | STM32 Pin | Notes |
| :--- | :--- | :--- |
| **i-BUS Signal (White/Yellow)** | **PA3 (RX2)** | High-speed 115200 serial packet stream |
| **VCC (Red)** | **5V (from ESC BEC)** | Receivers require 4.5V–6V |
| **GND (Black)** | **GND** | Common ground |

---

## 4. GPS Module (NEO-6M)

| GPS Pin | STM32 Pin | Function |
| :--- | :--- | :--- |
| **GPS TX** | **PA10 (RX1)** | Receives NMEA telemetry into STM32 |
| **GPS RX** | **PA9 (TX1)** | Transmits configuration commands to GPS |
| **GPS VCC** | **5V (from ESC BEC)** | GPS module power |
| **GPS GND** | **GND** | Common ground |

---

## 5. Auxiliary Peripherals

- **Active 5V Buzzer:** Positive to 5V, Negative to **PB2**.
- **Battery Voltage Sense (VBAT):** Voltage divider output to **PB0** (ADC1_IN8). Scaled to 110 in Betaflight.
- **Onboard Blue LED:** **PC13** (Active LOW).
