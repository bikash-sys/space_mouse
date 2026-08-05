# 3D Space Mouse

A USB HID 6‑DoF "space mouse" implementation for Raspberry Pi Pico using three MLX90393 magnetometers. The Pico exposes a generic gamepad HID (X, Y, Z, Rx, Ry, Rz) over USB and sends machine-readable sensor/debug output to the serial port for integration.

## Features
- HID Gamepad device (6 axes + buttons) via TinyUSB
- Sensor fusion from three MLX90393 magnetometers
- Serial debug output:
  - Raw sensor lines per sensor
  - Fused axis lines (Tx, Ty, Tz, Rx, Ry, Rz)
  - Machine-parsable CSV line starting with `AX,` for external consumers
- PlatformIO project for Raspberry Pi Pico (Arduino framework)


## Hardware
Required components:
- Raspberry Pi Pico (or compatible RP2040 board)
- 3 × MLX90393 magnetometer breakout boards (I2C)
- Wires / connectors, 3.3V power, ground

Sensor addresses in the code:
- ADDR_MAG1 = 0x0C (bottom)
- ADDR_MAG2 = 0x0D (top-left)
- ADDR_MAG3 = 0x0E (top-right)

Note: The address selection depends on your MLX90393 breakout solder jumpers or wiring. Ensure each sensor has a unique I2C address matching the defines above, or change the defines in code.

Typical wiring (per MLX90393 breakout):
- VCC -> 3.3V
- GND -> GND
- SDA -> Pico SDA (I2C0/1 depending on your wiring)
- SCL -> Pico SCL
- Ensure I2C pull-ups are present if your breakout doesn't include them.

Physical placement: sensors should be arranged around the knob (unit circle, ~120° apart):
- mag1: bottom
- mag2: top-left
- mag3: top-right

## Wiring
<img width="779" height="673" alt="Screenshot 2026-08-05 at 11 19 41 PM" src="https://github.com/user-attachments/assets/4507ca3d-b6a4-492c-a965-1e1c7cf49fb8" />
{im really sorry for such wiring but I had no option I cant find the exact magnetometer any where hope u get it }

## Software dependencies
- PlatformIO
- Platform: maxgerhardt/platform-raspberrypi (as referenced in platformio.ini)
- Board: pico
- Framework: Arduino (earlephilhower core)
- Libraries (PlatformIO `lib_deps`):
  - adafruit/Adafruit MLX90393
  - adafruit/Adafruit BusIO

## BOM
| S.No. | Component | Qty | Unit Price | Total | Purchase |
|---:|---|---:|---:|---:|---|
| 1 | Raspberry Pi Pico | 1 | $4.10 | **$4.10** | [Robu.in – Raspberry Pi Pico](https://robu.in/product/raspberry-pi-pico/) |
| 2 | SmartElex MLX90393 Triple-Axis Magnetometer | 3 | $3.00 | **$9.00** | [Robocraze – MLX90393](https://robocraze.com/products/smartelex-mlx90393-triple-axis-magnetometer-high-precision-magnetic-field-sensor) |
| 3 | 10 × 10 × 10 mm Neodymium Block Magnet | 3 | - | **$1.40** | [ElectroPi – Neodymium Magnet](https://www.electropi.in/10mm-x-10mm-x-10mm-10x10x10-mm-neodymium-block-magnet) |
| 4 | 3D Printed Base | 1 | — | — | Self printed |
| 5 | 3D Printed Knob | 1 | — | — | Self printed |

## Build & Flash (PlatformIO)
From the repository root:
- Build: pio run -e pico
- Upload: pio run -e pico -t upload
- Monitor serial: pio device monitor -e pico -b 115200

If using VSCode + PlatformIO, open the project, choose the `pico` environment and Build / Upload from the UI.

## Serial output format
The firmware prints several formats for debugging and external consumers:

1) Sensor raw format (human friendly):
   [0x0c] x=  12.3 y=  -4.1 z=  88.0   [0x0d] x=...  [0x0e] x=...

2) Fused axes (human friendly):
   Tx=  0.12 Ty= -0.03 Tz=  0.00 Rx=  0.00 Ry=  0.00 Rz=  0.01

3) Machine readable CSV (one line per cycle):
   AX,Tx,Ty,Tz,Rx,Ry,Rz,btn1,btn2
   Example:
   AX,0.0123,-0.0034,0.0000,0.0000,0.0000,0.0001,0,0

The HID report contains 6 int8 axes mapped to X,Y,Z,Rx,Ry,Rz and a buttons bitmask. Buttons are currently not wired in the code (TODO).

## Calibration & Tuning
- The firmware performs a quick calibration on boot: keep the knob still during boot (~200 ms) so offsets are captured.
- Key tuning constants are in the source (main.cpp):
  - GAIN_T / GAIN_R — per-axis gains for translation/rotation
  - DEAD_T / DEAD_R — deadzone thresholds
  - SMOOTH_TAU_S — smoothing time constant
  - OUTPUT_SCALE — scales normalized axis to int8 HID range
  - SIGN_AXIS — per-axis sign flips
- Adjust these constants if you see poor sensitivity, clipping, or excessive noise.

## Troubleshooting
- "One or more sensors failed to init" printed on serial:
  - Check wiring and power (3.3V vs 5V)
  - Verify each sensor's I2C address / solder jumpers
  - Confirm SDA/SCL connections and I2C pins on the Pico
  - Use an I2C scanner to confirm addresses on the bus
- USB/HID not recognized:
  - Ensure the Pico is in normal USB device mode and your USB cable supports data (not charge-only)
- No serial output:
  - Check Baud (115200) and correct serial port
  - Ensure the board is running and not stuck (use LED/debug prints)

## CAD Models
<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/6a231582-4df6-4898-b724-f7ee63f01a5b" />

<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/07f26fb4-b1c8-4aea-8485-6f3febabee55" />

## Assembly
The top part with magnetic housing goes into the base plate then connect the magnetometers and fix them in the gaps add magnets in housing and then just download the code folder open with vscode+platform io and just choose env pico and upload

## Build
<img width="1599" height="1200" alt="image" src="https://github.com/user-attachments/assets/fa500a0a-3d74-455a-9206-c3ba1f9b4e63" />

## Contributing
- Bug reports, PRs and improvements are welcome.
- Please describe hardware used and any wiring changes when submitting sensor-related fixes.


