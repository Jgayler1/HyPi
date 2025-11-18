
---

## `docs/tiers.md`

```markdown
# System Tiers

This project is structured in **tiers** so you don’t have to build the “fully loaded spaceship” just to get basic pH control.

---

## Tier 0 – pH-Only Controller (ESP32-only)

**Target user:**  
“I just want my reservoir around 5.8 without babysitting it.”

**Features:**

- 1× pH probe + pH interface board
- 1–2 peristaltic pumps:
  - pH down (required)
  - pH up (optional)
- ESP32 firmware:
  - pH target + hysteresis (e.g. 5.8 ± 0.1)
  - Dosing interval (time between allowed correction pulses)
  - Fixed pulse duration per dose
  - Simple safety:
    - Max pH down run time per hour/day
    - Optional “no-effect” fault if multiple doses don’t move pH

**No Raspberry Pi required.**  
You can set parameters via serial or hardcode them in firmware.

See: [`hardware_ph_only.md`](hardware_ph_only.md)

---

## Tier 1 – Full Controller (ESP32-only)

**Target user:**  
“I want pH + nutrients managed, plus more data.”

**Adds on top of Tier 0:**

- TDS/EC probe + interface board
  - TDS board is **power-gated** to avoid messing with pH readings
- DS18B20 solution temperature
- Optional DHT22 enclosure temp + RH
- 3× peristaltic pumps:
  - pH down
  - pH up
  - Nutrient
- ESP32 logic:
  - Dosing intervals for pH and nutrient
  - Per-pump pulse times (ms)
  - Hourly/day limits for pump run times
  - “No-effect” detection:
    - If repeated doses don’t change readings enough → trip fault
  - Optional float/leak input and master relay output

Still works **without** a Pi.  
The ESP32 alone can run the reservoir after it’s configured.

See: [`hardware_full_controller.md`](hardware_full_controller.md)

---

## Tier 2 – Optional GUI (Pi / PC)

**Target user:**  
“I want graphs, calibration wizards, and to stare at numbers all night.”

Uses a Raspberry Pi or any PC to run the Python GUI:

- Takes data from ESP32 over serial
- Shows:
  - Live pH, TDS, solution temp, RH, enclosure temp
  - 5 stacked graphs over time
- Lets you:
  - Set pH/TDS targets and hysteresis
  - Set dosing intervals and per-pump pulse durations
  - Manually fire dose pulses
  - Run 2-point pH calibration from the desktop

Recommended for:

- Development / tuning
- Long-term logging
- Lab-style setups

Not required for basic operation once firmware is configured.

See: [`pi_gui.md`](pi_gui.md)
