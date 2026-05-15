# Wind Tunnel Controller

A compact, standalone wind tunnel fan controller built with an ESP32-C3 round display and rotary knob. Turn the knob to set fan speed from 0–100% — the display updates in real time and the fan responds instantly via PWM.

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | VIEWE UEDX24240013-MD50E (ESP32-C3) |
| Display | 1.28" GC9A01A 240×240 round LCD |
| Input | Rotary encoder with push button |
| Fan | Noctua NF-R8 redux-1800 PWM (4-pin) |
| Power | USB 5V (controller) + 12V supply (fan) |

---

## Wiring

### Display & Encoder (built into VIEWE board)
| Signal | GPIO |
|---|---|
| SPI CLK | 1 |
| SPI MOSI | 0 |
| Display CS | 10 |
| Display DC | 4 |
| Display RST | 2 |
| Backlight | 8 |
| Encoder A | 6 |
| Encoder B | 7 |
| Encoder Button | 9 |

### Fan PWM Control
| Fan Wire | Colour | Connect To |
|---|---|---|
| Pin 1 | Black | GND (shared with ESP32) |
| Pin 2 | Yellow | 12V supply (+) |
| Pin 3 | Green | Not connected (tach) |
| Pin 4 | Blue | GPIO 21 / TX1 (PWM signal) |

> **Important:** The 12V supply negative must share a common GND with the ESP32 dev board.

---

## Software

Built with [ESPHome](https://esphome.io). The fan is driven at 25 kHz PWM — the standard for 4-pin PC fans. The rotary encoder maps 0–100 to 0–100% PWM duty cycle, updating the display and fan simultaneously on every tick.

### Setup

1. Install ESPHome:
   ```bash
   pip3 install esphome
   ```

2. Create a `secrets.yaml` in the project folder:
   ```yaml
   wifi_ssid: "YOUR_SSID"
   wifi_password: "YOUR_PASSWORD"
   ```

3. Compile and flash via USB:
   ```bash
   esphome run knob.yaml
   ```

4. For subsequent flashes, hold the rotary knob while plugging in USB to enter bootloader mode, then run:
   ```bash
   esptool --before default-reset --after hard-reset --baud 460800 \
     --port /dev/cu.usbmodem<PORT> --chip esp32c3 write-flash \
     -z --flash-size detect \
     0x10000 .esphome/build/knob-display/.pioenvs/knob-display/firmware.bin \
     0x0     .esphome/build/knob-display/.pioenvs/knob-display/bootloader.bin \
     0x8000  .esphome/build/knob-display/.pioenvs/knob-display/partitions.bin \
     0x9000  ~/.platformio/packages/framework-arduinoespressif32/tools/partitions/boot_app0.bin
   ```

---

## How It Works

```
Rotary Knob → ESPHome encoder sensor (0–100)
                    │
                    ├──► LEDC PWM output (GPIO 21, 25 kHz) ──► Noctua fan blue wire
                    │
                    └──► Display lambda redraws % value on GC9A01A screen
```

The display driver uses the `mipi_spi` platform with a custom C++ lambda that fills the screen and renders the percentage using a Roboto font at 70pt. The fan PWM output follows the 4-pin fan specification where duty cycle directly maps to fan speed.

---

## Notes

- `secrets.yaml` is excluded from this repo — never commit WiFi credentials
- The `mipi_spi` platform is required (the older `ili9xxx` platform has issues with newer ESPHome versions)
- USB logging uses `USB_SERIAL_JTAG` hardware UART — no UART0 conflict
- Fan runs at reduced max RPM on 9V; use 12V for full 1800 RPM
