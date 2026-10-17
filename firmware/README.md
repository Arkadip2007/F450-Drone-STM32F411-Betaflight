# Betaflight Firmware Directory

This directory contains both the **custom patched build** and the **official baseline releases** for the STM32F411CEU6 Black Pill.

## Subdirectories
- **`custom-builds/`**: Contains `betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex` with clone MPU-6050 support and remapped PB6/PB7 I2C routing.
- **`official-releases/`**: Contains stock upstream Betaflight release binaries (4.3.2 and 4.5.1) for reference.

## Flashing Recommendation
For the F450 drone using cheap MPU-6050 clone modules and analog 30A ESCs, use the custom build from `custom-builds/`.
