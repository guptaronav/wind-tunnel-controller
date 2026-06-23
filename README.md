# Wind Tunnel

A desktop open-circuit wind tunnel with fan-driven airflow, mist-based flow visualization, and a knob controller for fan speed. Everything needed to build one is in this repo: the 3D printable parts, the electronics wiring, and the ESP32-C3 firmware.

Rotate the knob to set fan speed — the round display shows the simulated mph of the test object (0–250 mph) in real time and the fan responds instantly over PWM. The mist maker injects visible fog into the airflow so you can see flow behavior through the clear test section.

---

## Repo layout

```
hardware/
  3d-printed-parts/   STL files for every printed part
  contraction-cone-profile.png   reference curve used to design the contraction cone
firmware/
  wind-tunnel.yaml     ESPHome config for the fan controller
  secrets.yaml         WiFi credentials (gitignored, create your own)
```

---

## Overview

Air is pulled through the tunnel by the fan at the exit end:

```
Honeycomb + screen inlet → Contraction cone → Test chamber → Diffuser → Fan
                                                  ↑
                                          Mist injected here
```

- **Honeycomb inlet / screen inlet** straighten and smooth incoming air before it accelerates through the contraction cone.
- **Contraction cone** narrows the flow into the test chamber, increasing velocity and evening out the profile.
- **Test chamber** is where models sit. A **top door** gives access without disassembling the tunnel. Mist is injected here (or just upstream of it) via the **manifold**, fed by an ultrasonic mist maker housed in the **mister enclosure**.
- **Diffuser** widens back out after the test chamber, recovering pressure before the fan.
- The **fan**, mounted at the exit, pulls air through the whole stack and is the only powered/controlled component.

---

## Wind tunnel basics

**What it's for.** A wind tunnel moves air past a stationary object instead of moving the object through still air. That makes it possible to study drag, lift, and flow separation on a scale model without building anything full size.

**Meshes and eddies.** An eddy is a swirling pocket of air that breaks away from the main flow, usually where flow separates around an edge or surface. Eddies make the airflow turbulent and uneven, which ruins flow visualization and skews test results. The honeycomb mesh forces air through a bundle of parallel cells, killing swirl and any sideways motion. The fine screen behind it breaks up the smaller eddies that survive and evens out the velocity across the whole cross-section, since faster-moving air meets more resistance passing through the mesh than slower air does.

**Venturi effect.** Air is effectively incompressible at these speeds, so the same volume has to pass through every cross-section per second. Narrow the duct and the air has to speed up to keep up, the same principle behind a Venturi tube. That's what the contraction cone does: it shrinks the cross-section ahead of the test chamber, accelerating and smoothing the flow before it reaches the model.

**Why a 5th-degree polynomial.** The contraction cone's wall profile (see [`contraction-cone-profile.png`](hardware/contraction-cone-profile.png)) is a quintic curve rather than a plain cone or a cubic, because a 5th-degree polynomial is the lowest degree that can satisfy six conditions at once: matching radius, slope, and curvature at both the inlet and the outlet. Matching slope keeps the wall tangent to the straight duct on each end with no kink. Matching curvature avoids a sudden change in the pressure gradient. Either discontinuity would trip the boundary layer into separating, adding turbulence right before the air reaches the test section.

---

## Bill of materials

### 3D printed (PLA, see [`hardware/3d-printed-parts/`](hardware/3d-printed-parts))

| Part | Qty | Notes |
|---|---|---|
| Base 1 / 2 / 3 | 1 each | Structural base, split into 3 prints |
| Contraction Cone 1 / 2 | 1 each | Two-piece cone, see profile reference image |
| Test Chamber 1 / 2 | 1 each | Two-piece test section |
| Diffuser 1 / 2 | 1 each | Two-piece diffuser |
| Honeycomb Inlet | 1 | Flow straightener, intake side |
| Honeycomb Outlet | 1 | Flow straightener, exit side |
| Screen Inlet | 1 | Mesh screen mount, intake side |
| Screen Outlet | 1 | Mesh screen mount, exit side |
| Top Door | 1 | Test chamber access hatch |
| Manifold | 1 | Distributes mist into the airflow |
| Mister Enclosure Bottom | 1 | Houses the ultrasonic mist maker |
| Mister Enclosure Lid | 1 | — |

### Electronics

| Part | Qty | Notes |
|---|---|---|
| VIEWE UEDX24240013-MD50E (ESP32-C3, round display + rotary encoder built in) | 1 | Fan controller |
| Noctua NF-R8 redux-1800 PWM fan (4-pin, 80mm) | 1 | ~12V, well under 0.1A draw |
| Ultrasonic mist maker module (fogger disc) | 1 | — |
| 12V DC power supply | 1 | — |
| USB 5V adapter | 1 | Powers the controller board |
| Hookup wire, 22–24 AWG | — | Fan and ground wiring |

---

## Wiring

### VIEWE board — internal pin mapping
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

### Fan connection
| Fan wire | Colour | Connect to |
|---|---|---|
| Pin 1 | Black | GND (shared with ESP32 and 12V supply −) |
| Pin 2 | Yellow | 12V supply + |
| Pin 3 | Green | Not connected (tachometer) |
| Pin 4 | Blue | GPIO 21 / TX1 (25 kHz PWM signal) |

> **Important:** The 12V supply negative and the ESP32 GND must share a common ground. Without this the fan will not respond to PWM.

---

## Software setup

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
   cd wind-tunnel-controller/firmware
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

## How the controller works

```
Rotary Knob (0–100)
       │
       ├──► LEDC PWM output (GPIO 21, 25 kHz) ──► Noctua fan blue wire
       │
       └──► Display redraws simulated mph (Roboto 70pt, GC9A01A via mipi_spi)
```

The rotary encoder maps 0–100 to a 0–100% PWM duty cycle at 25 kHz — the Intel specification for 4-pin PWM fans. That same 0–100 value is linearly scaled to 0–250 mph and rendered with a C++ lambda on the GC9A01A driver, representing the simulated speed of the test object rather than the tunnel's actual (much slower) airspeed — the same idea as a flight simulator showing airspeed independent of real moving air. Both the PWM output and the display update on every encoder tick with no polling loop.

---

## V1 status

V1 is built, assembled, and tested end to end — the mist maker produces visible flow and the knob controller drives the fan. Known improvements for V2:

- Screw mounts for the diffuser fan (currently not secure enough)
- Visual redesign of the test section
- A proper enclosure for the controller and wiring, instead of sitting loose

---

## Notes

- `secrets.yaml` is gitignored — never commit WiFi credentials
- Use `mipi_spi` platform (not the deprecated `ili9xxx`) for reliable display rendering
- Logger uses `USB_SERIAL_JTAG` to keep GPIO 20/21 free for UART use
- Fan runs below rated RPM on 9V; use 12V for full 1800 RPM
- After flashing, the ESP32 can be powered from any USB adapter — no computer needed
