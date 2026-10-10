# Toshiba AW-LINK (TCC-LINK / AB Bus) Protocol Specification

> **Language / Język:** [🇬🇧 English](PROTOCOL-AW-LINK.md) | [🇵🇱 Polski](PROTOCOL-AW-LINK.pl.md)

This document details the reverse-engineered physical and application layers of the 2-wire **AB bus** (AW-LINK) used by **Toshiba Estia** heat pumps (Series 4 & Series 5 Hipower, e.g., Hydrobox `HWS-P805XWHT6-E` and Outdoor Unit `HWS-P805HR-E`).

---

## 1. Physical Layer (PHY)

* **Wiring:** 2-wire unpolarized communication bus (terminals marked **A** and **B** on the hydrobox PCB terminal block `CN02` / `CN03`).
* **Nominal Line Voltage:** ~15–18 V DC (powers wired remote controllers such as `HWS-AMS54E`).
* **Baud Rate:** `2400 baud`
* **Data Format:** `8N1` (8 data bits, no parity, 1 stop bit).
* **Bus Transceiver Topology:** Half-duplex. Preamble sync bytes `0xF0 0xF0` and postamble `0xA0` delimit raw bus frames.

---

## 2. Bus Addressing Scheme

| Node Type | Primary Address | Secondary Address | Broadcast / Special |
|---|---|---|---|
| **Master Hydrobox** | `0x70` | `0x71` | `0xFF` |
| **Main Remote Controller** | `0x60` | `0x61` | — |
| **Sub-Remote Controller** | `0x68` | `0x69` | — |

---

## 3. Frame Structure & Checksum Algorithm

A decoded payload frame adheres to the following layout:

```text
[LEN] [SRC] [DST] [ROUTING] [SEQ/CMD] [PAYLOAD ...] [CHECKSUM]
```

* **`LEN`**: Total payload length (including `LEN` byte itself and `CHECKSUM`).
* **`SRC`**: Sender address (`0x60` for remote, `0x70` for hydrobox).
* **`DST`**: Recipient address (`0x70` for hydrobox, `0x60` for remote).
* **`CHECKSUM`**: 8-bit sum modulo 256 of all bytes from `LEN` up to the byte preceding `CHECKSUM`:
  $$\text{CHECKSUM} = \left(\sum_{i=0}^{N-1} \text{byte}[i]\right) \pmod{256}$$

---

## 4. Key Discovery: Zone 1 Water Temperature Byte Decoding

> [!IMPORTANT]
> **Bug in Upstream Projects:** Previous community implementations read the Zone 1 flow water temperature from `raw[14]`, producing `-16°C` errors.
> **Correct Byte:** Zone 1 supply water temperature ($T_{WO}$) is located in **byte 13** (`raw[13]`):
> $$T_{WO}\ [^\circ\text{C}] = \frac{\text{raw}[13]}{2} - 16$$

---

## 5. Control Frames (Remote Controller $\to$ Hydrobox)

### 5.1 Strefa 1 (Zone 1) Power State (`0x21`)
* **TURN OFF:**
  ```text
  0B:60:70:E0:01:21:02:DF
  ```
* **TURN ON (HEAT):**
  ```text
  0B:60:70:E0:01:21:03:E0
  ```

### 5.2 Auto Weather Curve Mode (`0x24:01`)
* **MANUAL / FIXED TARGET:**
  ```text
  0C:60:70:E0:01:24:01:00:E2
  ```
* **AUTO WEATHER CURVE:**
  ```text
  0C:60:70:E0:01:24:01:01:E3
  ```

### 5.3 Water Target Temperature Command (`0x23:02`)
Injects target setpoint ($T_{target} \in [20, 55]^\circ\text{C}$):
```text
0F:60:70:E0:01:23:02:<ENC>:66:A2:<ENC>:<CHK>
```
Where:
$$\text{ENC} = (T_{target} + 16) \times 2$$
$$\text{CHK} = \text{Checksum}(0x0F, \dots, \text{ENC})$$

### 5.4 Silent Mode / Night Mode (`0x24:04`)
* **SILENT MODE OFF:**
  ```text
  0C:60:70:E0:01:24:04:00:E5
  ```
* **SILENT MODE ON:**
  ```text
  0C:60:70:E0:01:24:04:04:E9
  ```

---

## 6. Installer Field Settings (DN Codes) Protocol

Universal frame structure for querying and modifying installer EEPROM settings directly over the AB bus without using the remote's physical buttons.

### 6.1 DN Code Query Frame (`0x1A:F1`)
Remote request:
```text
0D:60:70:E0:41:1A:F1:<DN_HEX>:00:<CHK>
```
Hydrobox response (`0x1A`):
```text
0C:70:60:E0:80:1A:<DN_HEX>:<VAL_HEX>:<CHK>
```

### 6.2 DN Code Write Frame (`0x1C`)
Writes setting directly to EEPROM:
```text
F0:F0:0C:60:70:E0:01:1C:<DN_HEX>:<DATA_HEX>:<CHK>:A0
```
Master ACK response:
```text
0A:70:60:E0:80:A1:DB
```

*Example Live Capture (Setting Curve Shift DN 27 = +2 K):*
```text
Remote TX: 0C:60:70:E0:01:1C:27:02:02
Master RX: 0A:70:60:E0:80:A1:DB (ACK)
```

### 6.3 Exit Installer Menu
```text
0A:60:70:E0:00:62:1C
0A:60:FF:E0:88:62:33
```
