# Hardware & Transceiver Guide (Makusets v3.2 & DIY)

> **Language / Język:** [🇬🇧 English](HARDWARE-GUIDE.md) | [🇵🇱 Polski](HARDWARE-GUIDE.pl.md)

This guide covers physical installation, power supply optimization, and flashing procedures for connecting ESP8266/ESP32 to the Toshiba AB line.

---

## 1. Makusets v3.2 Transceiver Board Overview

The dedicated **Makusets v3.2** board is optimized for Toshiba / Carrier AB bus interfacing:
* **Bus Protection & Rectification:** `MB10S` full-bridge rectifier eliminates polarity sensitivity on the AB line.
* **Buck Converter:** `K7803-500` high-efficiency step-down converter steps down the ~15–18 V bus voltage to clean 3.3 V DC.
* **Microcontroller:** ESP-12F (ESP8266) with onboard blue status LED connected to **GPIO2**.
* **UART Bus Pins:**
  * **TX:** `GPIO12` (transmits to bus via optical/transistor driver)
  * **RX:** `GPIO13` (receives from bus comparator)
* **USB Interface:** CH340 USB-to-UART converter for initial programming.

---

## 2. Power Selection Jumper (`U5`)

> [!CAUTION]
> **NEVER connect both USB and AB bus power simultaneously with the jumper on the wrong setting!**

* **Jumper Position `USB`:** Power is drawn exclusively from the USB 5V rail (via onboard AMS1117 3.3V). Use this mode on your workbench when flashing initially over USB.
* **Jumper Position `AB`:** Power is drawn directly from the 2-wire AB communication line of the hydrobox. Disconnect the USB cable before setting the jumper to `AB`.

---

## 3. Critical Hardware Quirks & Solutions

### 3.1 Initial USB Flashing (CH340 Auto-Reset Quirk)
The CH340 circuit on Makusets v3.2 does not connect RTS/DTR lines to the ESP's GPIO0 and RST pins.
* **To enter bootloader:**
  1. Hold down the **BOOT** button.
  2. Press and release the **RESET** button.
  3. Release the **BOOT** button.
  4. Run `esphome run toshiba-estia-r410a.yaml` (or `esptool.py write_flash`).
* **After flashing:** You must physically press **RESET** or power-cycle the board. The esptool "hard reset via RTS" command has no physical effect.

### 3.2 Over-The-Air (OTA) Brownout on AB Line Power
When powered by the AB bus through the `K7803-500` buck converter, peak RF current during flash erase (~200 mA at 20 dBm Wi-Fi TX power) causes a brief 3.3V voltage sag, triggering an ESP watchdog reset or OTA timeout.

**The Fix:**
Limit Wi-Fi TX power in the ESPHome configuration:
```yaml
wifi:
  output_power: 14.0dBm
```
This saves ~100 mA during RF bursts, providing 100% reliable OTA flashing over the AB bus.

---

## 4. Hydrobox Terminal Connection

Connect a 2-wire twisted-pair cable (e.g., standard alarm or cat5 cable) between the Makusets board `A`/`B` terminals and the Toshiba Hydrobox main board:
* Terminals: **`CN02` / `CN03` (A, B)** located on the main hydrobox PCB.
* Can be wired in parallel with existing remote controllers (`HWS-AMS54E`).
* Line is unpolarized (A and B can be swapped without issue).
