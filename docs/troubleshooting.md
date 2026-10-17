# Betaflight Black Pill Troubleshooting & Diagnostic Guide

## 1. Arming Prevention Flags (Why the drone won't arm)

Type `status` in the Betaflight CLI to view active arming lock flags:

| Arming Flag | Root Cause | Solution |
| :--- | :--- | :--- |
| **`CLI`** | The CLI tab is currently open | Normal behavior; close CLI or disconnect to arm |
| **`LOAD`** | CPU utilization is too high (>75%) | Run `set pid_process_denom = 8` in CLI to limit PID loop to 1kHz |
| **`CALIB`** | Gyro is calibrating or drone is moving | Keep the drone motionless on a flat surface upon power-up |
| **`RXLOSS`** | No signal from radio receiver | Check UART2 RX connection (`PA3`) and receiver binding |
| **`THROTTLE`** | Throttle stick is above minimum | Lower throttle stick to minimum (<1050 µs) |
| **`NOPREARM`** | Prearm switch not engaged | Toggle the configured pre-arm switch |

---

## 2. DFU Mode Driver Issues on Windows

If Betaflight Configurator does not detect the Black Pill when plugged in with `BOOT0` pressed:
1. Open Device Manager and find `STM32 BOOTLOADER` with a yellow triangle.
2. Download and run **ImpulseRC Driver Fixer** — it will detect the board and install the correct DFU driver automatically.
3. Alternatively, open **Zadig**, select `Options > List All Devices`, choose `STM32 BOOTLOADER`, and replace the driver with `WinUSB`.

---

## 3. Only One Motor Spins on the Bench

If only one motor spins when testing with transmitter in Angle mode without props:
- **This is normal PID I-term windup.** Without propellers, the flight controller tilts slightly, and the I-term accumulator continuously increases power to the lower side attempting to level the craft.
- Tilt the frame by hand to verify all four motors react dynamically.
- Test individual motors safely using the Configurator Motors tab with propellers removed.
