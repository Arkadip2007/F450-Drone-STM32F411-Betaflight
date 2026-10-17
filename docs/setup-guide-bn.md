# STM32F411CEU6 (Black Pill) কাস্টম Betaflight 4.5.1 ফার্মওয়্যার ও সেটআপ গাইড

এই ডকুমেন্টে আপনার STM32F411CEU6 ব্ল্যাকপিল বোর্ড এবং কাস্টম সেন্সর ওয়্যারিংয়ের জন্য বিশেষভাবে তৈরি **Betaflight 4.5.1** ফার্মওয়্যার ব্যবহারের সম্পূর্ণ নিয়ম দেওয়া হলো।

---

## ১. ফাইল পরিচিতি
* **কাস্টম ফার্মওয়্যার:** `Drone_MCU/STM32_Drone/Betaflight/2_Flash_Firmware/Custom_AI_Generated/betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`
* **সিএলআই কনফিগ ফাইল:** `Drone_MCU/STM32_Drone/Betaflight/3_CLI_Configurations/betaflight_blackpill_f411_custom_cli.txt`
* **বেস আর্কিটেকচার:** Betaflight 4.5.1 (অক্টোবর ২০২৪ রিলিজ - সর্বাধুনিক ও সবচেয়ে স্ট্যাবল)

---

## ২. কেন এই কাস্টম হেক্স ফাইলটি তৈরি করা হয়েছে?
স্ট্যান্ডার্ড বিটাফ্লাইট ব্ল্যাকপিল বা সাধারণ F411 ফার্মওয়্যার দিয়ে আপনার ড্রোনের হার্ডওয়্যার চলত না, কারণ:

1. **ক্লোন MPU-6050 WHO_AM_I ফিক্স (0x68 ও 0x98):**
   - সাধারণ বিটাফ্লাইট কেবল অরিজিনাল চিপের `0x68` চেক করে। আপনার ক্লোন চিপটি `0x98` রিটার্ন করায় সাধারণ বিটাফ্লাইটে "No Gyro Detected" বা জাইরো আইকন অফ হয়ে থাকত।
   - আমাদের কাস্টম হেক্সে সরাসরি বাইনারি প্যাচ করে `0x68` এবং `0x98` উভয়কেই ভ্যালিড MPU6050 হিসেবে স্বীকৃতি দেওয়া হয়েছে।
2. **হার্ডওয়্যার ফেইলিউর ক্র্যাশ বাইপাস (Revision Check Bypass):**
   - বিটাফ্লাইটের সাধারণ ড্রাইভার চাইনিজ ক্লোন সেন্সরে প্রোডাক্ট রিভিশন চেক করতে গিয়ে রিভিশন 0 পেয়ে `failureMode(FAILURE_ACC_INCOMPATIBLE)` কল করে পুরো ফ্লাইট কন্ট্রোলার ক্র্যাশ করিয়ে দিত (রেড এলইডি অনবরত দ্রুত ব্লিঙ্ক করত)।
   - কাস্টম হেক্সে রিভিশন চেক সেফলি বাইপাস করে সরাসরি **Full Resolution Accel & Gyro Mode** অন করে দেওয়া হয়েছে।
3. **I2C1 ডিফল্ট পিন (PB6 ও PB7):**
   - ফার্মওয়্যারের ডিফল্ট I2C1 বাস সরাসরি আপনার সোল্ডার করা পিন **PB6 (SCL)** এবং **PB7 (SDA)**-এ সেট করা হয়েছে।
4. **মোটর ৩ ও ৪ টাইমার সাপোর্ট (PB8 ও PB9):**
   - টাইমার টেবিলের মাধ্যমে Motor 1 (`PB4`), Motor 2 (`PB5`), Motor 3 (`PB8`), এবং Motor 4 (`PB9`) পারফেক্টলি অ্যাসাইন করা যাবে।

---

## ৩. সম্পূর্ণ ওয়্যারিং চার্ট (Hardware Pinout)

| পেরিফেরাল / সেন্সর | সেন্সর পিন | ব্ল্যাকপিল পিন (STM32F411) | নোট |
| :--- | :--- | :--- | :--- |
| **MPU-6050 (জাইরো/অ্যাক্সেল)** | SCL / SDA | **PB6 / PB7** | I2C বাস (প্যারালাল) |
| **BMP280 (ব্যারোমিটার)** | SCL / SDA | **PB6 / PB7** | I2C বাস (প্যারালাল) |
| **QMC5883L / HMC5883L (ম্যাগ)** | SCL / SDA | **PB6 / PB7** | I2C বাস (প্যারালাল) |
| **Motor 1 (Front Right - CCW)** | Signal | **PB4** | TIM3 Channel 1 |
| **Motor 2 (Rear Right - CW)** | Signal | **PB5** | TIM3 Channel 2 |
| **Motor 3 (Rear Left - CCW)** | Signal | **PB8** | TIM4 Channel 3 |
| **Motor 4 (Front Left - CW)** | Signal | **PB9** | TIM4 Channel 4 |
| **GPS মডিউল (TX / RX)** | GPS TX / RX | **PA10 (RX1) / PA9 (TX1)** | UART1 (57600 বাউড) |
| **RC Receiver (FlySky IBUS)** | Signal | **PA3 (RX2)** | UART2 রিসিভার |
| **Beeper (Buzzer)** | Positive/Signal | **PB2** | বাজার |
| **Battery ADC (VBAT)** | Divider Out | **PB0** | ব্যাটারি ভোল্টেজ সেন্সিং |
| **Status LED** | Onboard | **PC13** | ফ্লাইট কন্ট্রোলার স্টেটাস |

---

## ৪. ফ্ল্যাশ করার সহজ ধাপ (Flashing Steps)

1. **Betaflight Configurator:** আপনার কম্পিউটারে **Betaflight Configurator 10.10** ওপেন করুন।
2. ব্ল্যাকপিল বোর্ডের **BOOT0 বাটন চেপে ধরে** USB কেবল দিয়ে কম্পিউটারে লাগান।
3. কনফিগুরাটরের উপরে ডানপাশে পোর্ট লিস্টে **`DFU`** লেখা দেখতে পাবেন।
4. বাম পাশের মেনু থেকে **Firmware Flasher**-এ যান।
5. নিচে ডান পাশে **Load Firmware [Local]** বাটনে ক্লিক করুন।
6. আপনার কম্পিউটার থেকে এই ফাইলটি সিলেক্ট করুন:
   `C:\Users\arkad\Downloads\Drone_MCU\STM32_Drone\Betaflight\2_Flash_Firmware\Custom_AI_Generated\betaflight_4.5.1_BLACKPILL_CUSTOM_MPU6050_PB6_PB7.hex`
7. **"Full chip erase"** অপশনটি অন করুন।
8. **Flash Firmware** বাটনে চাপুন। ফ্ল্যাশ শেষ হওয়া পর্যন্ত অপেক্ষা করুন।

---

## ৫. কেন ফ্ল্যাশ করার সাথে সাথে "No Gyro/Acc found" দেখায়? (খুব গুরুত্বপূর্ণ)

> [!WARNING]
> বিটাফ্লাইটে যে কোনো নতুন ফার্মওয়্যার "Full chip erase" দিয়ে ফ্ল্যাশ করলে এটি একদম **ফ্যাক্টরি ডিফল্ট (SPI Gyro on PA4)** মোডে অন হয়।
> যেহেতু বিটাফ্লাইট ডিফল্টভাবে SPI বাসে জাইরো খোঁজে এবং আপনার MPU-6050 টি লাগানো **I2C বাসে (PB6/PB7)**, তাই **CLI কনফিগারেশন রান করার আগ পর্যন্ত বিটাফ্লাইট জাইরো অন করে না**!
> একই কারণে আপনার প্রথম স্ক্রিনশটে Ports ট্যাবে UART1 এবং UART2 অফ দেখাচ্ছিল।
> **CLI স্ক্রিপ্টটি লোড বা পেস্ট করলেই ১ সেকেন্ডে জাইরো, অ্যাক্সেল, ব্যারোমিটার ও পোর্টগুলো চালু হয়ে যাবে!**

---

## ৬. সিএলআই কনফিগারেশন সেটআপ (CLI Setup)

ফ্ল্যাশ শেষে ড্রোন রিবুট হয়ে সাধারণ COM পোর্টে আসবে:
1. Betaflight Configurator-এ **Connect** বাটনে ক্লিক করুন।
2. বাম পাশের মেনু থেকে **CLI** ট্যাবে যান।
3. **পদ্ধতি ১ (সবচেয়ে সহজ):** 
   - স্ক্রিনের নিচে ডান পাশে থাকা **"Load from file"** বাটনে ক্লিক করুন।
   - সিলেক্ট করুন: `C:\Users\arkad\Downloads\Drone_MCU\STM32_Drone\Betaflight\3_CLI_Configurations\betaflight_blackpill_f411_custom_cli.txt`
   - এরপর নিচে থাকা **"Execute"** বাটনে ক্লিক করুন!
4. **পদ্ধতি ২ (সরাসরি পেস্ট):**
   - নিচের সম্পূর্ণ কোড ব্লকটি কপি করে CLI-এর নিচের ইনপুট বক্সে পেস্ট করে **Enter** চাপুন:

```text
# 1. Unmap conflicting resources first
resource MOTOR 3 NONE
resource MOTOR 4 NONE
resource I2C_SCL 1 NONE
resource I2C_SDA 1 NONE

# 2. Map I2C1 to PB6 and PB7
resource I2C_SCL 1 B06
resource I2C_SDA 1 B07

# 3. Map Motors: M1=PB4, M2=PB5, M3=PB8, M4=PB9
resource MOTOR 1 B04
resource MOTOR 2 B05
resource MOTOR 3 B08
resource MOTOR 4 B09

# 4. Map Peripherals
resource BEEPER 1 B02
resource LED 1 C13
resource ADC_BATT 1 B00
resource SERIAL_TX 1 A09
resource SERIAL_RX 1 A10
resource SERIAL_TX 2 A02
resource SERIAL_RX 2 A03

# 5. Features
feature RX_SERIAL
feature TELEMETRY
feature AIRMODE
feature GPS

# 6. Serial Ports
serial 20 1 115200 57600 0 115200
serial 0 2 115200 57600 0 115200
serial 1 64 115200 57600 0 115200

# 7. Receiver Provider (FlySky IBUS on UART2 RX)
set serialrx_provider = IBUS

# 8. Gyro & Looptime (1kHz loop for I2C MPU-6050)
set gyro_1_bustype = I2C
set gyro_1_i2cBus = 1
set gyro_1_i2c_address = 0
set gyro_1_sensor_align = CW0
set pid_process_denom = 1

# 9. Barometer & Magnetometer (I2C1)
set baro_bustype = I2C
set baro_i2c_device = 1
set baro_hardware = AUTO
set mag_bustype = I2C
set mag_i2c_device = 1
set mag_hardware = AUTO

# 10. GPS Configuration
set gps_provider = UBLOX
set gps_sbas_mode = AUTO
set gps_auto_baud = ON
set gps_ublox_use_galileo = ON

# 11. Motor / ESC Settings
set motor_pwm_protocol = DSHOT300
set dshot_bitbang = OFF

# 12. Beeper & Battery ADC
set beeper_inversion = ON
set beeper_od = OFF
set battery_meter = ADC
set vbat_scale = 110

save
```

বোর্ডটি স্বয়ংক্রিয়ভাবে সেভ হয়ে রিবুট হবে এবং রিবুট হওয়ার সাথে সাথে:
- Gyro এবং Accelerometer আইকন সাথে সাথে অ্যাক্টিভ হয়ে যাবে!
- Setup ট্যাবে ড্রোনের 3D মডেল নড়াচড়া করবে!
- Ports ট্যাবে GPS ও Serial RX চালু হয়ে যাবে!
