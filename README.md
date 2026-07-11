# stm32_greenhouse

# Connected Greenhouse | STM32 Sensor Station

A DIY connected greenhouse monitoring system built around an **STM32** microcontroller. It reads environmental data (light, soil moisture, temperature and humidity) and displays it in real time on an OLED screen, while also logging it over UART.

This project was built for **Fête de la Science** (French science fest), under the theme **"Saveur Savante"**, as part of a workshop demonstrating how electronics is used in modern agriculture. The workshop paired this sensor station with remote-controlled tractor robots to illustrate two complementary sides of agri-tech: **sensing the environment** (this project) and **mechanized field tools**.

## Project Goal

Show visitors, in a simple and visual way, how sensors can capture the physical conditions of soil and surrounding air — the same kind of data real smart-farming systems rely on to optimize crop growth, irrigation, and resource use.

## Hardware

| Component | Role | Interface |
|---|---|---|
| STM32L476RG (Nucleo) | Main controller | — |
| LDR (photoresistor) | Ambient light level | ADC2 (Channel 5) |
| Soil moisture sensor (HW-390) | Soil humidity | ADC2 (Channel 6) |
| DHT11 | Air temperature & humidity | Custom bit-banged GPIO protocol |
| SSD1306 OLED display | Live data display (I2C) | I2C3 |
| USART2 (115200 baud) | Serial data logging | UART |
| TIM6 | Microsecond timing for DHT11 protocol | Timer |

## How It Works

1. **Light sensing** : the LDR is read via the ADC and converted to a percentage (0–100%).
2. **Soil moisture sensing** : the raw ADC reading is calibrated between two reference values (`AIR_VALUE` = sensor in open air, `WATER_VALUE` = sensor in water) and mapped to a 0–100% humidity scale, clamped to avoid out-of-range values.
3. **Temperature & humidity (DHT11)** : since the STM32 HAL has no built-in DHT11 driver, a custom driver was written from scratch:
   - `TIM6` is used as a free-running microsecond timer to precisely measure the sensor's timing-based signal (the DHT11 protocol encodes bits as variable-length pulses).
   - The GPIO pin is dynamically switched between output (to send the start signal) and input with pull-up (to read the response), following the DHT11 single-wire protocol.
   - A checksum is verified on the 40 received bits before accepting the reading.
4. **Display** : readings are shown cyclically on a 128x64 **SSD1306 OLED** screen (soil humidity → light level → temperature), using a slightly modified version of a popular open-source SSD1306 driver library adapted for this project's font and screen layout needs.
5. **Logging** : all raw and processed values are also sent over **UART2** (115200 baud) for debugging or connection to a PC/serial monitor.

## Configuration Notes

- **Soil sensor calibration**: `AIR_VALUE` and `WATER_VALUE` in `main.c` should be recalibrated for your own sensor — dip it in water and leave it in open air to get accurate raw ADC readings, then update the constants accordingly.
- **DHT11 pin**: connected on `PA10`, configured with an internal pull-up for the read phase.
- **Timing**: `TIM6` is configured with a prescaler to tick at 1 MHz (1 tick = 1 µs), which is essential for correctly decoding the DHT11 protocol.

## Credits

- SSD1306 OLED driver: based on an open-source library, lightly modified for this project (custom fonts, display sequence).
- DHT11 driver: custom implementation for STM32 HAL, using TIM6 for precise microsecond timing.

---
