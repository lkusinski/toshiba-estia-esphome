# Toshiba Estia Installer DN Codes Reference

> **Language / Język:** [🇬🇧 English](DN-CODES-REFERENCE.md) | [🇵🇱 Polski](DN-CODES-REFERENCE.pl.md)

This document maps essential Toshiba Estia (Series 4 & 5) installer settings (DN codes) that can be queried and modified over the AB bus.

---

## 1. Primary Heating & Curve Parameters

| DN (Hex) | DN (Dec) | Parameter Name | Default | Range | Unit | Description |
|---|---|---|---|---|---|---|
| `0x27` | **DN 27** | Heating Curve Shift / Offset | `0x00` (0) | -5 to +5 | K / °C | Parallel vertical shift of heating water curve. |
| `0x28` | **DN 28** | Heating Target Water Temp (Warm) $T_{W1}$ | `0x19` (25) | 20 to 55 | °C | Target water flow temp at warm outdoor boundary. |
| `0x29` | **DN 29** | Heating Target Water Temp (Cold) $T_{W2}$ | `0x2A` (42) | 25 to 55 | °C | Target water flow temp at freezing outdoor boundary. |
| `0x2A` | **DN 30** | Outdoor Boundary Temp (Warm) $T_{E1}$ | `0x11` (17) | 10 to 25 | °C | High outdoor cutoff temperature. |
| `0x2B` | **DN 31** | Outdoor Boundary Temp (Cold) $T_{E2}$ | `0xF1` (-15)| -20 to 5 | °C | Freezing design outdoor temperature. |

---

## 2. Thermostat Hysteresis & Circulation Pump Control

| DN (Hex) | DN (Dec) | Parameter Name | Default | Range | Unit | Description |
|---|---|---|---|---|---|---|
| `0x19` | **DN 25** | Thermostat ON Hysteresis | `0x03` (3) | 1 to 10 | K | Temp drop below target before restarting compressor. |
| `0x1A` | **DN 26** | Thermostat OFF Hysteresis | `0x05` (5) | 1 to 10 | K | Temp overshoot above target before stopping compressor. |
| `0x24` | **DN 36** | Circulation Pump Speed | `0x06` (6) | 1 to 7 | Step | Nominal speed profile for Grundfos UPM2 K pump. |
| `0x27` | **DN 39** | Pump Post-Run Delay | `0x08` (8) | 1 to 30 | min | Pump circulation duration after compressor shutdown. |

---

## 3. Power Limits & Backup Heater

| DN (Hex) | DN (Dec) | Parameter Name | Default | Range | Unit | Description |
|---|---|---|---|---|---|---|
| `0x44` | **DN 68** | Outdoor Unit Current Limit | `0x0F` (15) | 5 to 20 | A | Current ceiling for compressor in emergency/PV modes. |
| `0x09` | **DN 09** | Backup Heater Delay | `0x4B` (75) | 10 to 120| min | Time delay before engaging auxiliary 6 kW electric heater. |

---

## 4. How to Read & Write DN Codes via Home Assistant

The ESPHome component exposes:
* `number.toshiba_estia_r410a_estia_przesuniecie_krzywej_dn_27` (Interactive slider for DN 27).
* ESPHome API services:
  * `esphome.toshiba_estia_r410a_write_dn_code(code: 39, value: 2)`
  * `esphome.toshiba_estia_r410a_query_dn_code(code: 39)`
