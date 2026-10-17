# Betaflight CLI Configurations

This directory contains the custom CLI script required to map all pins, enable communication ports, and configure loop times on the STM32F411 Black Pill.

## File: `betaflight_blackpill_f411_custom_cli.txt`

### How to Apply:
1. Connect the flight controller via USB to Betaflight Configurator.
2. Navigate to the **CLI** tab on the left sidebar.
3. Open `betaflight_blackpill_f411_custom_cli.txt`, copy its full contents, and paste into the text box.
4. Press **Enter**. The commands will execute in sequence.
5. Type `save` and press **Enter**. The board will reboot with all settings applied.

### What it Configures:
- Unmaps conflicting default resources (`MOTOR 3/4`, `I2C1`)
- Maps I2C1 to `PB6` / `PB7`
- Maps Motors 1–4 to `PB4`, `PB5`, `PB8`, `PB9`
- Enables serial RX on `PA3` (FlySky i-BUS)
- Enables GPS on `PA9`/`PA10` (UART1 @ 115200)
- Sets `pid_process_denom = 8` to fix 81% CPU load
- Configures analog PWM protocol (400Hz) for 30A SimonK ESCs
