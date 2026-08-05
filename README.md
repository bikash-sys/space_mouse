# 3D_Space_Mouse

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

## Software dependencies
- PlatformIO
- Platform: maxgerhardt/platform-raspberrypi (as referenced in platformio.ini)
- Board: pico
- Framework: Arduino (earlephilhower core)
- Libraries (PlatformIO `lib_deps`):
  - adafruit/Adafruit MLX90393
  - adafruit/Adafruit BusIO

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
  - PlatformIO sets a test USB PID (0x1209/0x0001). Do not distribute firmware with the test PID; change to a vendor-approved PID when shipping.
- No serial output:
  - Check Baud (115200) and correct serial port
  - Ensure the board is running and not stuck (use LED/debug prints)

## CAD Models
![alt text](image.png)
![alt text](image-1.png)
## Contributing
- Bug reports, PRs and improvements are welcome.
- Please describe hardware used and any wiring changes when submitting sensor-related fixes.

## License
No license file is included in the repository. If you plan to use or distribute this project, add an appropriate LICENSE file (e.g., MIT, Apache-2.0) to clarify permissions.
