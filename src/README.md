# Betaflight Source Code Driver Modifications

This directory contains source-level modifications made to Betaflight for this build.

## File: `gyrodev.c`

### Purpose
Contains the patch for the MPU-6050 gyroscope driver. Authentic InvenSense sensors return `0x68` on WHO_AM_I register `0x75`. Counterfeit / clone chips commonly return `0x98` or `0x70`.

In stock Betaflight, the detection check fails, leaving gyro and accelerometer disabled. In this modified driver:

```c
// Gyroscope WHO_AM_I detection whitelist
#define MPU_ADDRESS_CLONE_98 0x98
#define MPU_ADDRESS_CLONE_70 0x70

// If chip returns 0x68, 0x98, or 0x70, it is accepted as MPU-6050
```

This source file serves as a reference for developers compiling Betaflight from source.
