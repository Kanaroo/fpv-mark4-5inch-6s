# Dronii — 5" FPV Quadcopter Build

A 6S 5-inch FPV quadcopter built and configured from scratch: frame assembly, FC/ESC soldering, ELRS binding, GPS integration, analog VTX setup, and Betaflight tuning.

This repo documents the build so it can be rebuilt or repaired from the config alone — component list, wiring map, and the full Betaflight `diff all`.

---

## Specs

| | |
|---|---|
| **Frame** | iFlight Mark4 (5") |
| **FC / ESC** | SpeedyBee F405 V5 stack (STM32F405) |
| **Motors** | iFlight XING2 2207 1855KV (6S) |
| **Receiver** | SpeedyBee Nano ELRS |
| **Transmitter** | RadioMaster TX12 (ELRS) |
| **Camera** | Caddx Ratel Pro |
| **VTX** | Rush Max Solo (2.5W, SmartAudio) |
| **Goggles** | Skyzone Cobra X V4 (analog) |
| **GPS** | HGLRC M100 (with compass) |
| **Battery** | 6S LiPo |
| **Firmware** | Betaflight 4.5.2 — target `SPEEDYBEEF405V5` |

---

## Wiring / UART map

| Port | Device | Function | Baud |
|---|---|---|---|
| USB VCP | — | Configuration/MSP | 115200 |
| UART1 | — | Unused | — |
| UART2 | — | Unused | — |
| UART3 | Rush Max Solo VTX | Peripheral — TBS SmartAudio | 115200 |
| UART4 | HGLRC M100 GPS | Sensor input — GPS | 57600 |
| UART5 | ESC | Sensor input — ESC telemetry | 57600 |
| UART6 | SpeedyBee Nano ELRS | Serial RX | — |
| I2C | Compass | On the GPS module | — |

> Wiring photos: [Google Drive album](https://drive.google.com/drive/folders/1DKZdByoKoDQ1DKYEFXLckaTX7_ePrfbe?usp=drive_link)

---

## Betaflight configuration

The full config is in [`betaflight-diff-all.txt`](./betaflight-diff-all.txt) — restore it by pasting into the CLI and running `save`.

**Motor output**
- DShot300 with **bidirectional DShot** enabled → RPM filtering active
- `yaw_motors_reversed = ON` (props-out configuration)
- ESC telemetry enabled (`feature ESC_SENSOR`)

**Filtering**
- Dynamic notch reduced to a single notch (`dyn_notch_count = 1`) with a tight Q (`dyn_notch_q = 500`) — viable because RPM filtering already handles the motor noise peaks, and it keeps filter delay down.

**Failsafe & GPS Rescue**
- Failsafe procedure: `AUTO-LAND`, throttle 1100
- GPS Rescue armed on a dedicated switch
- Minimum 10 satellites before rescue will engage; sanity checks on
- Galileo constellation enabled alongside GPS
- Arming permitted without a GPS fix (so line-of-sight flights aren't blocked indoors/under cover)

**Logging**
- Blackbox at 1/2 sample rate, `debug_mode = GYRO_SCALED` — enough resolution for filter and PID analysis without saturating the write rate.

**Video**
- PAL, OSD canvas 30×16
- Custom `vtxtable`: 6 bands × 8 channels, 4 power levels (25 mW / 500 mW / 1 W / 2.5 W)
- Running band A / channel 1 (5865 MHz) at 1 W

---

## Switch layout

| Switch | Mode | Range |
|---|---|---|
| AUX1 | **ARM** | 1700–2100 |
| AUX2 | **ANGLE** | 900–1300 |
| AUX2 | **HORIZON** | 1300–1700 |
| AUX4 | **BEEPER** | 1700–2100 |
| AUX7 | **BLACKBOX** | 1300–1700 |
| AUX8 | **GPS Rescue** | 1700–2100 |

ANGLE and HORIZON share AUX2 as a three-position switch: low = ANGLE, mid = HORIZON, high = ACRO.

---

## OSD

Configured for GPS-assisted flying rather than pure freestyle:

- Battery: average cell voltage
- Link: RSSI, RSSI dBm, link quality
- GPS: speed, satellite count, home direction, home distance, flight distance
- Flight mode, VTX channel, crosshair, artificial horizon, warnings

---

## Repo contents

```
├── README.md
├── betaflight-diff-all.txt    # Full CLI config dump
└── docs/
    └── wiring.md              # Detailed wiring diagram
```

Build and assembly photos are hosted separately: [Google Drive album](https://drive.google.com/drive/folders/1DKZdByoKoDQ1DKYEFXLckaTX7_ePrfbe?usp=drive_link)

---

## Rebuilding this config

1. Flash Betaflight 4.5.2 to the `SPEEDYBEEF405V5` target
2. Open the CLI, paste the contents of `betaflight-diff-all.txt`
3. Run `save`
4. Recalibrate the accelerometer and compass — the calibration values in the dump are specific to this airframe
5. Rebind the ELRS receiver
