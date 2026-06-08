# Wind Tunnel Controller

A standalone fan speed controller built with an ESP32-C3 round display module. Rotate the knob to set fan speed — the circular display shows the current percentage in real time, and the fan responds instantly over PWM.

Built with [ESPHome](https://esphome.io) and designed to run headlessly off a USB adapter with no computer required.

---

## Demo

> Rotate knob → display updates → fan speed changes

---

## Hardware

| Component | Part |
|---|---|
| Controller | VIEWE UEDX24240013-MD50E (ESP32-C3) |
| Display | 1.28" GC9A01A 240×240 round LCD (built in) |
| Input | Rotary encoder with push button (built in) |
| Fan | Noctua NF-R8 redux-1800 PWM (4-pin) |
| Power — controller | USB 5V wall adapter |
| Power — fan | 12V DC supply |

---

## Wiring

### VIEWE Board — Internal Pin Mapping
| Signal | GPIO |
|---|---|
| SPI CLK | 1 |
| SPI MOSI | 0 |
| Display CS | 10 |
| Display DC | 4 |
| Display RST | 2 |
| Backlight PWM | 8 |
| Encoder A | 6 |
| Encoder B | 7 |
| Encoder Button | 9 |

### Fan Connection
| Fan Wire | Colour | Connect To |
|---|---|---|
| Pin 1 | Black | GND (shared with ESP32 and 12V supply −) |
| Pin 2 | Yellow | 12V supply + |
| Pin 3 | Green | Not connected (tachometer) |
| Pin 4 | Blue | GPIO 21 / TX1 (25 kHz PWM signal) |

> **Important:** The 12V supply negative and the ESP32 GND must share a common ground. Without this the fan will not respond to PWM.

---

## Software Setup

### Requirements
- Python 3.8+
- ESPHome

```bash
pip3 install esphome
```

### Configuration

1. Clone this repo:
   ```bash
   git clone https://github.com/guptaronav/wind-tunnel-controller.git
   cd wind-tunnel-controller
   ```

2. Create `secrets.yaml` (not committed — keep this private):
   ```yaml
   wifi_ssid: "YOUR_SSID"
   wifi_password: "YOUR_PASSWORD"
   ```

3. Compile:
   ```bash
   esphome compile wind-tunnel.yaml
   ```

4. Flash — hold the rotary knob while plugging in USB to enter bootloader mode, then:
   ```bash
   esptool --before default-reset --after hard-reset --baud 460800 \
     --port /dev/cu.usbmodem<PORT> --chip esp32c3 write-flash \
     -z --flash-size detect \
     0x10000 .esphome/build/wind-tunnel/.pioenvs/wind-tunnel/firmware.bin \
     0x0     .esphome/build/wind-tunnel/.pioenvs/wind-tunnel/bootloader.bin \
     0x8000  .esphome/build/wind-tunnel/.pioenvs/wind-tunnel/partitions.bin \
     0x9000  ~/.platformio/packages/framework-arduinoespressif32/tools/partitions/boot_app0.bin
   ```

5. After first flash, OTA updates work wirelessly — just run `esphome run wind-tunnel.yaml`.

---

## How It Works

```
Rotary Knob (0–100)
       │
       ├──► LEDC PWM output (GPIO 21, 25 kHz) ──► Noctua fan blue wire
       │
       └──► Display redraws percentage (Roboto 70pt, GC9A01A via mipi_spi)
```

The rotary encoder maps 0–100 to a 0–100% PWM duty cycle at 25 kHz — the Intel specification for 4-pin PWM fans. The display renders the percentage using a C++ lambda on the GC9A01A driver. Both update on every encoder tick with no polling loop.

---

## Notes

- `secrets.yaml` is gitignored — never commit WiFi credentials
- Use `mipi_spi` platform (not the deprecated `ili9xxx`) for reliable display rendering
- Logger uses `USB_SERIAL_JTAG` to keep GPIO 20/21 free for UART use
- Fan runs below rated RPM on 9V; use 12V for full 1800 RPM
- After flashing, the ESP32 can be powered from any USB adapter — no computer needed
