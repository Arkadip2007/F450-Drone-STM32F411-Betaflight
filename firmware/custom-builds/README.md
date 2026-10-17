# Custom Patched Betaflight Firmware

- **Filename:** `betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`
- **Target MCU:** STM32F411CEU6 (Black Pill)
- **Base Version:** Betaflight 4.5.1

## Key Patches Applied
1. **MPU-6050 Clone Whitelist:** Whitelists silicon revision ID `0x98` in register `0x75` (`WHO_AM_I`).
2. **I2C1 Remapping:** Configured to initialize I2C1 on `PB6` (SCL) and `PB7` (SDA) without resource conflict.
3. **Timer Allocation:** Preserves `TIM3` (PB4, PB5) and `TIM4` (PB8, PB9) for 4 independent hardware PWM motor channels.

## How to Flash
Use Betaflight Configurator in DFU mode with **Full chip erase** enabled.
