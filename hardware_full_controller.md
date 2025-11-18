docs/hardware_full_controller.md# Tier 1: Full Controller Hardware (pH + TDS + Temp + 3 Pumps)

This is the **“full rig”** version:  
pH, TDS, temperature, and three dosing pumps, all managed by an ESP32.

---

## Core Components

- **Controller**
  - 1× ESP32 dev board

- **Sensors**
  - pH probe + pH interface board (BNC input, analog output)
  - TDS/EC probe + TDS interface board
    - TDS board’s VCC is switched (relay/MOSFET) to avoid interfering with pH
  - 1× DS18B20 waterproof temperature sensor (solution temp)
  - 1× DHT22 (or similar) for enclosure temp + RH (optional)

- **Pumps**
  - 3× 12V peristaltic pumps:
    - pH down
    - pH up
    - Nutrient

- **Relays / Switching**
  - 1× 4-channel relay module for pumps
  - 1× additional relay (or MOSFET) for **TDS board power gating**
  - 1× master relay or MOSFET to cut **all 12V to pumps** on fault

- **Safety Inputs**
  - 1× float switch for “overflow” level in reservoir (optional but encouraged)
  - 1× leak/moisture sensor tray under the system

- **Power**
  - 12V supply sized for pumps (and valve if used)
  - 5V buck for ESP32 and relay logic

---

## Key Wiring Points

### Pump Relays

- Relay module VCC/GND → 5V + GND
- Relay inputs:
  - IN1 → ESP32 `RELAY_PH_DOWN`
  - IN2 → ESP32 `RELAY_PH_UP`
  - IN3 → ESP32 `RELAY_NUTRIENT`
- Each pump’s positive line goes through its relay’s COM/NO.

### TDS Board Power Gating

- 5V (or 3.3V, per your TDS board) → **COM** on small relay or MOSFET
- **NO** → TDS board VCC
- TDS board GND → common ground
- TDS board analog output → ESP32 `TDS_PIN`

ESP32 pin (e.g. `TDS_ENABLE_PIN`):

- Drives the relay coil / MOSFET gate
- Firmware does:
  - Turn TDS on
  - Wait (e.g. 300 ms)
  - Read ADC
  - Turn TDS off

This keeps pH readings clean.

### pH Board

Same as Tier 0:

- VCC → ESP32 3.3/5V (check board)
- GND → common
- Analog out → `PH_PIN` (e.g. GPIO 34)

### DS18B20

- Data → ESP32 pin defined as `ONE_WIRE_BUS`
- 4.7k pull-up from data to 3.3V
- GND/VCC as usual

### DHT22

- Data → `DHT_PIN`
- VCC → 3.3/5V per sensor
- GND → common

### Master Relay

Used to cut 12V (or 24V) to **all pumps/valves**:

- 12V supply → COM (master relay)
- NO → 12V rail feeding pump relays
- Coil is driven via:
  - Transistor/MOSFET + ESP32 `MASTER_ENABLE_PIN`
  - And optionally in series with float/leak switches

Firmware can **drop MASTER_ENABLE_PIN low** on fault to hard-kill pumps.

---

## Safety Scenarios Covered

- Max run-time per pump (per hour / per day)
- “No-effect” fault if pH/TDS doesn’t change after multiple corrections
- Overflow/leak fault from float/moisture sensors
- Master relay to cut all pump power on fault

ESP32 outputs a `FAULT,<reason>` line over serial so the GUI can display what went wrong.

---

## Recommended Build Order

1. Build and validate **Tier 0** pH-only behavior with a single pump.  
2. Add:
   - DS18B20
   - DHT22
3. Add:
   - Nutrient pump + TDS board (with TDS power gating)
4. Add:
   - Float/leak sensor
   - Master relay
5. Finally:
   - Connect the Python GUI (Tier 2) and start tuning in a real reservoir.
