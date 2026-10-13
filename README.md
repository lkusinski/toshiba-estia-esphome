# ❄️ Toshiba Estia ESPHome AB-Bus Integration

> **Language / Język:** [🇬🇧 English](README.md) | [🇵🇱 Polski](README.pl.md)

[![ESPHome Version](https://img.shields.io/badge/ESPHome-2026.8%2B-teal.svg)](https://esphome.io/)
[![Hardware](https://img.shields.io/badge/Hardware-Makusets%20v3.2%20(ESP--12F)-orange.svg)](docs/HARDWARE-GUIDE.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Protocol](https://img.shields.io/badge/Protocol-AW--LINK%20(2400%208N1)-purple.svg)](docs/PROTOCOL-AW-LINK.md)
[![GitHub Fork](https://img.shields.io/badge/Fork-makusets%2Fesphome--toshiba--ab-green.svg)](https://github.com/lkusinski/esphome-toshiba-ab)

> [!NOTE]
> **Complete Turnkey Solution & Upstream Merge:**
> This repository provides a complete, turnkey Home Assistant & ESPHome integration suite for Toshiba Estia Series 4 & 5 heat pumps (including packages, Lovelace dashboards, and protocol documentation). It includes an enhanced, standalone build of the `toshiba_ab` component.  
> Our enhancements and fixes have been officially merged into the upstream repository via [**Pull Request #245**](https://github.com/makusets/esphome-toshiba-ab/pull/245) (resolving [Issue #242](https://github.com/makusets/esphome-toshiba-ab/issues/242)).

Autonomous **ESPHome C++ firmware** and **Home Assistant package** for direct interfacing with **Toshiba Estia** air-to-water heat pumps (Series 4 & 5 Hipower, R410A / R32) over the 2-wire unpolarized **AB communication bus (AW-LINK)**.

Tested and verified on live hardware:
* **Hydrobox (Indoor Unit):** `HWS-P805XWHT6-E`
* **Outdoor Unit:** `HWS-P805HR-E` (8 kW Hipower, R410A)
* **Wired Remote Controller:** `HWS-AMS54E`
* **Transceiver Board:** Makusets v3.2 (powered directly from the AB line)

---

## ⚙️ Key Features

1. **Native ESPHome Climate Entity (`climate` in C++) with 0.5 K Resolution:**
   * Eliminates the need for fragile `climate_template` or mode synchronization automations in Home Assistant.
   * Directly controls heat pump operating modes (`OFF`, `HEAT`, `AUTO` weather compensation) and water setpoint target (20–55 °C) via native frames (`0x21`, `0x24:01`, `0x23:02`).
   * **Natively supports 0.5°C steps:** The AW-LINK bus natively accepts half-degree granularity even though the physical remote only allows full degrees! Exposes `target_temp_step: 0.5` in Home Assistant for refined radiator curve tuning (e.g. 31.5°C).
2. **Hardware Inverter Modulation Limit (Silent Mode 30 Hz):**
   * Eliminates short cycling during mild weather caused by the aggressive factory 42–48 Hz startup surge.
   * Capping the inverter frequency to **30.0 Hz** via the native Silent Mode frame (`0x24:04`) lets the heat pump gently ramp into heating without tripping the +2 K water limit. Extends runs to continuous **30+ minute cycles** down to **10–11 Hz** with daily **COP reaching 7.7**.
3. **Zone 1 Water Temperature Decoding (Byte 13):**
   * Addresses the persistent `-16°C` error in earlier community implementations (which read byte 14). Supply temperature is decoded from **byte 13** ($T = \text{raw}[13]/2 - 16$).
4. **Installer Field Settings (DN Codes) via Bus:**
   * Direct EEPROM parameter control over the AB line using reverse-engineered installer frames (`0x1C` write and `0x1A:F1` query).
   * Remotely adjust thermostat hysteresis (DN 25/26), Grundfos pump speed (DN 36), compressor current limit (DN 68), and heating curve shift (DN 27) without touching the wall controller!
5. **Complete Refrigeration Circuit Telemetry & Remote Temp:**
   * High pressure ($P_D$) & low pressure ($P_S$) in kPa.
   * Discharge temp ($T_D$), suction temp ($T_S$), heat exchanger temp ($T_E$), inverter fin temp ($T_{HS}$), and liquid temp ($T_C$).
   * Ambient temperature from the wired wall remote controller decoded directly from bus packets (`0x81`).
   * Compressor modulation (Hz), PMV expansion valve steps, and DC fan RPM.
6. **On-Device COP & Energy Metering:**
   * Real-time thermal and electrical power estimation computed directly on the microcontroller.
   * Daily thermal energy produced (kWh), electrical energy consumed (kWh), and average daily COP without overloading Home Assistant's database.
7. **Reliable AB Bus Power & OTA Stability:**
   * Solved OTA brownout resets on bus-powered boards by tuning Wi-Fi TX power (`output_power: 14.0dBm`).
   * Visual bus activity feedback via onboard blue LED on **GPIO2**.
   * Hardware safety read-only interlock (`read_only_switch: ALWAYS_ON` on boot) cuts TX line at the microcontroller level.
8. **Room Temperature Feedback & Room Cut-off Thermostat:**
   * Direct target room temperature adjustment (`input_number.estia_oczekiwana_temperatura_w_domu`, e.g. 21.0°C) with dynamic heating water curve compensation ($K_p$).
   * **Dual-Hysteresis Room Cut-off:** When the house exceeds the target temperature by the turn-off threshold (>= +0.2 K), the heating zone completely powers down (`hvac_mode: off`), shutting down both compressor and circulating pump to eliminate short cycling during mild weather. Automatically restarts (`hvac_mode: heat`) once the home cools down (<= -0.1 K).
9. **Physical Electricity Submeter Validation & Drift Calibration:**
   * Built-in Home Assistant audit helpers and sensors (`input_number.estia_licznik_fizyczny_stan`) comparing physical DIN-rail electricity submeter readings against bus telemetry.
   * Real-time calculation of absolute difference (kWh), relative error (%), and calibration factor $k = \Delta E_{\text{fiz}} / \Delta E_{\text{HA}}$ to verify auxiliary consumption (Grundfos pump, hydrobox PCB, crankcase heater).
10. **Operational Oversight, Manual Override Protection & Compressor Cycle Monitoring:**
   * **5-Item FIFO Action Log:** Distinguishes physical wired remote adjustments from automated adjustments, protects manual overrides, and provides an instant one-click return to automation.
   * **Anti-Cycling Telemetry:** Real-time `history_stats` sensors tracking compressor starts over 1-hour, 24-hour, and daily intervals.

---

## 📁 Repository Structure

```text
├── README.md                          # Project documentation and quick-start guide
├── LICENSE                            # MIT License
├── docs/
│   ├── PROTOCOL-AW-LINK.md            # Reverse-engineered frame specs, timings & checksums
│   ├── DN-CODES-REFERENCE.md          # Installer code mappings (DN 09, 25, 26, 27, 36, 68...)
│   └── HARDWARE-GUIDE.md              # Board wiring, jumpers, USB vs AB power, flashing notes
├── esphome/
│   ├── toshiba-estia-r410a.yaml       # Ready-to-use ESPHome configuration
│   ├── secrets.yaml.example           # Wi-Fi and encryption key template
│   └── components/
│       └── toshiba_ab/                # Enhanced C++ custom component
│           ├── __init__.py
│           ├── climate.py
│           ├── automation.h
│           ├── toshiba_ab.cpp
│           └── toshiba_ab.h
└── home-assistant/
    ├── packages/
    │   └── toshiba_estia_optymalizator.yaml  # Clean HA package (PV boost, G12w, summer auto-off)
    └── dashboards/
        └── lovelace_pompa_ciepla.json        # Production Lovelace dashboard config
```

---

## 🚀 Quick Start Guide

### 1. Hardware Preparation
1. Set the power jumper `U5` on your Makusets v3.2 board to **`USB`**.
2. Connect the board via micro-USB to your computer.
3. Hold the **BOOT** button, press and release **RESET**, then release **BOOT** to enter the bootloader.

### 2. Configure & Flash ESPHome
1. Copy `esphome/secrets.yaml.example` to `esphome/secrets.yaml` and fill in your Wi-Fi credentials and API encryption key.
2. Compile and flash the firmware:
   ```bash
   esphome run esphome/toshiba-estia-r410a.yaml
   ```
3. Press **RESET** physically on the board after flashing completes (the CH340 RTS line does not reset the ESP automatically).

### 3. Connect to the Heat Pump
1. Disconnect the USB cable.
2. Move the `U5` power jumper to **`AB`**.
3. Turn off the heat pump breaker for safety.
4. Connect the board's `A` and `B` terminals to the hydrobox `CN02`/`CN03` terminal block in parallel with the wall remote controller.
5. Restore the heat pump breaker. The board boots up within seconds, indicated by the blue LED on GPIO2.

### 4. Home Assistant Integration
1. Home Assistant will discover the device automatically via the native ESPHome integration.
2. Copy `home-assistant/packages/toshiba_estia_optymalizator.yaml` into your `/config/packages/` folder.
3. Import `home-assistant/dashboards/lovelace_pompa_ciepla.json` into your Lovelace dashboard.

---

## 📊 Dashboard Preview

The provided Lovelace dashboard includes:
* Native Thermostat card linked directly to `climate.toshiba_estia_r410a_estia_ogrzewanie_strefy_1`.
* Silent Mode toggle (`switch.toshiba_estia_r410a_estia_tryb_cichy_agregatu`).
* Live Defrost indicator (`binary_sensor.toshiba_estia_r410a_estia_odszranianie_defrost`).
* Installer DN 27 heating curve parallel shift slider.
* On-device daily thermal/electric energy meters & daily COP.
* Mini-graph cards for compressor frequency modulation (Hz), high/low pressures ($P_D$, $P_S$), and flow/return temperatures ($T_{WO}$, $T_{WI}$).

---

## 🇵🇱 Podsumowanie po polsku (Polish Summary)

Projekt stanowi kompletny i przetestowany na żywym sprzęcie system integracji pomp ciepła **Toshiba Estia (seria 4 i 5 Hipower)** z **Home Assistant** przez dwuprzewodową magistralę AB (AW-LINK).

**Co rozwiązano:**
* **Natywny termostat C++:** ESPHome sam wystawia pełną encję `climate` ze wszystkimi trybami i nastawą temperatury.
* **Naprawiony odczyt zasilania C.O.:** woda zasilania pobierana jest z bajtu 13 zamiast błędnego bajtu 14 (co powodowało odczyt `-16°C` w projektach społeczności).
* **Zdalna zmiana kodów DN:** uniwersalny protokół zapisu i odczytu kodów instalatora (m.in. histereza DN 25/26, bieg pompy Grundfos DN 36, limit prądu agregatu DN 68).
* **Stabilne zasilanie z magistrali:** zoptymalizowana moc nadajnika Wi-Fi (14 dBm) eliminuje resety i pozwala na bezprzewodowe aktualizacje OTA bezpośrednio w kotłowni.
* **Czysty pakiet Home Assistant:** brak syntetycznych szablonów, optymalizacja pod taryfy G12w i nadwyżki fotowoltaiki (PV) oraz automatyczny postój w ciepłe dni (Summer Switch-Off).

---

## 📜 Credits & Acknowledgments
* Built upon foundational work by [makusets/esphome-toshiba-ab](https://github.com/makusets/esphome-toshiba-ab), the pioneering physical transceiver research by [issalig/toshiba_air_cond](https://github.com/issalig/toshiba_air_cond), and the TCC-Link ESPHome implementation by [muxa/esphome-tcc-link](https://github.com/muxa/esphome-tcc-link).
* Tested, adapted, and documented on a residential Toshiba Estia Series 5 Hipower installation.
* Upstream fork with verified patches: [lkusinski/esphome-toshiba-ab](https://github.com/lkusinski/esphome-toshiba-ab).
