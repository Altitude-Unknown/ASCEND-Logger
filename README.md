# ASCEND Logger

Firmware and circuit board design for the ASCEND Logger, the air-quality and
flight data logger that ASCEND teams fly. It records GPS position and time,
pressure and particulate readings to a microSD card as CSV that follows the
[ASCEND data standard](https://ascendflight.org/data/standard/).

> **Status: in development.** The firmware here is bench and prototype code.
> It is not yet a finished, flight-ready logger.

## Hardware (board v11)

- SAMD21 microcontroller with USB
- u-blox GPS on I2C (address 0x42) with a W3011 chip antenna
- MS5611 barometer on I2C (address 0x76)
- microSD card on SPI (CS D10), formatted FAT32
- Optional Adafruit PMSA003I particulate sensor on I2C (address 0x12)
- No radio

Known issues for the next board revision:

- GPS reception is weak. The antenna needs more ground-plane clearance.
- The schematic lists U1 as an MS5607. The fitted part is an MS5611.

## What's in this repository

| Folder | Contents |
| --- | --- |
| [`ASCEND_Logger_Diagnostic/`](ASCEND_Logger_Diagnostic/) | Bench diagnostic for the v11 board (I2C scan, GPS, barometer), the v11 schematic PDF, and the EAGLE schematic and board files in `cad-review/` |
| [`ASCEND_UAS/`](ASCEND_UAS/) | Earlier logging firmware: GPS, pressure and PM at 1 Hz to SD, with a LoRa backup link. It targets YELLOWSTONE or a Feather M0. See its README for the CSV format and bench results |
| [`ASCEND_Ground/`](ASCEND_Ground/) | Ground receiver for the `ASCEND_UAS` LoRa packets, logging the same CSV rows |
| [`ASCEND_SD_Verify/`](ASCEND_SD_Verify/) | SD card write and readback check |
| [`libraries/ASCENDCore/`](libraries/ASCENDCore/) | Shared packet, CSV, radio and file handling |
| [`bench-captures/`](bench-captures/) | Raw serial captures from bench tests |
| `check_capture.py` | Checks a captured CSV log against the format |

## Build

Install [arduino-cli](https://arduino.github.io/arduino-cli/), then run from the
repository root:

```sh
# v11 board diagnostic (Altitude Unknown SAMD board package)
arduino-cli compile --fqbn AltitudeUnknown:samd:altitude_yellowstone_m0 ASCEND_Logger_Diagnostic

# Logging firmware and ground receiver (Adafruit SAMD core)
arduino-cli lib install --no-deps 'Adafruit PM25 AQI Sensor@1.1.1'
arduino-cli compile --fqbn adafruit:samd:adafruit_feather_m0 --libraries libraries ASCEND_UAS
arduino-cli compile --fqbn adafruit:samd:adafruit_feather_m0 --libraries libraries ASCEND_Ground
```

Upload with `arduino-cli upload -p <port> --fqbn <board> <sketch>`, using the port
shown by `arduino-cli board list`. Library versions are listed in
[`ASCEND_UAS/README.md`](ASCEND_UAS/README.md).

Check a captured log:

```sh
python3 check_capture.py bench-captures/bench-capture.txt
```

## History

This code was first developed in the
[YELLOWSTONE](https://github.com/Altitude-Unknown/YELLOWSTONE) repository on the
`ascend/uas-datalogger` branch, and moved here with its history.
