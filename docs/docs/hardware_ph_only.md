# Tier 0: pH-Only Controller Hardware

This is the **cheapest, simplest** useful build:  
maintain pH in a hydroponic reservoir with **one ESP32 and 1–2 pumps**.

---

## Core Components

Approximate / generic parts list (no specific brands required):

- **Controller**
  - 1× ESP32 dev board (e.g., ESP32-DevKitC style)

- **pH Measurement**
  - 1× pH probe (standard BNC hydroponic probe)
  - 1× pH interface board (pH → analog voltage, 0–3.3V range)

- **Dosing Pumps**
  - 1× 12V peristaltic pump for pH down (required)
  - 1× 12V peristaltic pump for pH up (optional but recommended)

- **Relays / Drivers**
  - 1× 4-channel relay board (5V input, logic-level controlled)
    - You only *need* 2 channels (pH up/down), but the extra channels give you upgrade paths.

- **Power**
  - 1× 12V DC supply, sized for your pumps (~2–3A is usually sufficient)
  - 1× 5V buck converter for ESP32 + relay board **OR** 5V USB for ESP32 and 12V only for pumps
  - Fuses are strongly recommended on the 12V line

- **Safety (Optional but Recommended)**
  - 1× float or overflow switch in the reservoir
  - 1× leak/moisture sensor under the system
  - 1× master relay or MOSFET to cut 12V pump power in case of fault

- **Misc**
  - Tubing for pump suction/discharge
  - Mounting hardware, project box, etc.

---

## Basic Wiring

High-level wiring only; adjust to your exact parts.

### Power

- 12V PSU → pumps + relay module **VCC** (if your relay module uses 12V coils)  
- Or 5V relay module:
  - 12V PSU → pumps
  - 5V buck → ESP32 5V/USB and relay board VCC

**Grounds must be common**:

- ESP32 GND
- Relay board GND
- pH board GND
- 12V PSU GND (through the relay module ground)

### Pumps

Each pump:

- Pump + line → COM / NO contacts of its relay
- Relay module:
  - `IN1` → ESP32 GPIO for pH down
  - `IN2` → ESP32 GPIO for pH up
  - `VCC` → 5V
  - `GND` → ground

Use **NO (Normally Open)** so pumps are off by default and only run when commanded.

### pH Board

- pH board **VCC** → ESP32 3.3V / 5V (check your board)
- pH board **GND** → ESP32 GND
- pH board analog output → ESP32 ADC pin (configured in firmware, e.g. GPIO 34)

### Safety Inputs (Optional)

- Float / leak sensors → ESP32 GPIOs, configured as digital inputs with pull-ups
- In firmware, any active sensor trips a **fault** that:
  - Shuts off per-pump relays
  - Can drop a master relay if used

---

## Firmware Behavior (pH-Only)

See `firmware/esp32_ph_only/` for the actual code, but in summary:

- Reads pH via ADC, converts using:
  - `pH = offset - slope * voltage` (2-point calibration)
- If pH > target + hysteresis:
  - Runs pH down pump for `pulseDownMs`
  - Waits at least `doseIntervalMs` before next correction
- If pH < target − hysteresis:
  - Optionally runs pH up pump for `pulseUpMs`
- Enforces:
  - Max pH down run time per hour/day
  - Optional “no effect after N doses” fault

---

## Bring-up Checklist

1. Wire ESP32 + pH board + single pump (pH down)  
2. Flash pH-only firmware  
3. Confirm:
   - Serial output shows pH values changing with buffer solutions
   - Pump only runs when pH is above target + hysteresis
4. Tune:
   - `pulseDownMs` based on your pump’s flow and reservoir volume
   - `doseIntervalMs` so each dose has time to fully mix before the next decision
5. Add pH up pump and additional safety once base behavior is validated.
