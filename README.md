# HyPi
Automated RDWC system
# Hy-Pi Hydroponic Controller

DIY hydroponic controller built around an **ESP32** and a handful of peristaltic pumps.

- Automatically maintains **pH** (and optionally **nutrient strength / TDS**)
- Drives **pH up**, **pH down**, and **nutrient** pumps
- Supports **DS18B20** solution temperature, **DHT22** enclosure temp/RH (full build)
- Optional **Raspberry Pi / PC GUI** for live graphs, calibration, logging

Commercial gear that does something comparable is easily $1–2k+.  
This project is designed to be **buildable for a couple hundred bucks** in parts if you shop smart.

> **Goal:**  
> ESP32 is the *brains* for control and safety.  
> Raspberry Pi (or PC) is *optional eye candy* (GUI, graphs, logging).

---

## Project Status

- ✅ ESP32 firmware: pH + TDS + temp, pumps, basic safety, calibration  
- ✅ Python GUI: Tkinter + matplotlib, manual dosing, auto run, calibration, graphing  
- ✅ Architecture for “Tier 0” pH-only controller (ESP32-only)  
- 🔜 ESP32-only web UI (no Pi needed for UI)

---

## Architecture Overview

The project is split into **three tiers** so people can build what they actually need:

- **Tier 0 – pH-only Controller (ESP32 only)**
  - 1× pH probe + interface board
  - 1–2 peristaltic pumps (pH down, optional pH up)
  - Basic auto dosing around a setpoint with hysteresis
  - Simple safety limits (max dosing per hour / “no effect” lockout)
  - No Pi required

- **Tier 1 – Full Controller (ESP32 only)**
  - pH + TDS + solution temperature (+ optional RH / enclosure temp)
  - 3 peristaltic pumps (pH up, pH down, nutrients)
  - Dosing intervals & per-pulse times
  - Safety: max run-time, no-effect protection, overflow/moisture input
  - TDS board is power-gated to avoid messing up pH readings

- **Tier 2 – Optional GUI (Pi / PC)**
  - Python/Tk GUI with:
    - Live readings and status
    - Graphs (pH, TDS, temps, RH)
    - pH setpoints & hysteresis controls
    - Dosing intervals & pulse time controls
    - Manual dose buttons
    - 2-point GUI-assisted pH calibration
  - Runs nicely on a Pi 5 or modest PC; Pi Zero W can do it headless via VNC.

See [`docs/tiers.md`](docs/tiers.md) for more detail.

---

## Repo Layout

Suggested structure (you can adjust as you like):

```text
.
├── firmware/
│   ├── esp32_ph_only/          # Tier 0: simple pH-only controller
│   └── esp32_full/             # Tier 1: full pH+TDS+temp controller
├── gui/
│   └── pi_tk_gui/              # Tier 2: Python/Tk/matplotlib GUI
├── docs/
│   ├── tiers.md
│   ├── hardware_ph_only.md
│   ├── hardware_full_controller.md
│   ├── firmware_esp32.md
│   └── pi_gui.md
└── LICENSE
