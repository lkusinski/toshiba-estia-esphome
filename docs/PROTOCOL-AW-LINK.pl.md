# Specyfikacja Protokołu Toshiba AW-LINK (TCC-LINK / Magistrala AB)

> **Język / Language:** [🇬🇧 English](PROTOCOL-AW-LINK.md) | [🇵🇱 Polski](PROTOCOL-AW-LINK.pl.md)

Dokument opisuje zdekodowaną warstwę fizyczną oraz aplikacyjną 2-żyłowej magistrali komunikacyjnej **AB** (AW-LINK) stosowanej w pompach ciepła **Toshiba Estia** (seria 4 i seria 5 Hipower, np. Hydrobox `HWS-P805XWHT6-E` oraz jednostka zewnętrzna `HWS-P805HR-E`).

---

## 1. Warstwa Fizyczna (PHY)

* **Okablowanie:** 2-żyłowa niespolaryzowana magistrala (zaciski oznaczone **A** i **B** na listwie hydroboksu `CN02` / `CN03`).
* **Napięcie robocze:** ~15–18 V DC (służy również do zasilania pilota przewodowego `HWS-AMS54E`).
* **Prędkość transmisji (Baud Rate):** `2400 baud`.
* **Format ramki UART:** `8N1` (8 bitów danych, brak parzystości, 1 bit stopu).
* **Topologia:** Half-duplex. Bajty synchronizacji `0xF0 0xF0` otwierają ramkę, a `0xA0` zamyka transmisję.

---

## 2. Adresacja na Magistrali

| Typ Urządzenia | Adres Główny | Adres Zapasowy | Rozgłoszeniowy (Broadcast) |
|---|---|---|---|
| **Hydrobox (Master)** | `0x70` | `0x71` | `0xFF` |
| **Główny Pilot Ścienny (Remote)** | `0x60` | `0x61` | — |
| **Drugi Pilot (Sub-Remote)** | `0x68` | `0x69` | — |

---

## 3. Struktura Ramki i Suma Kontrolna

Ramka danych po odrzuceniu preambuły i postambuły ma postać:

```text
[LEN] [SRC] [DST] [ROUTING] [SEQ/CMD] [DANE ...] [CHECKSUM]
```

* **`LEN`**: Całkowita długość danych (włącznie z samym bajtem `LEN` oraz bajtem `CHECKSUM`).
* **`SRC`**: Adres nadawcy (`0x60` dla pilota, `0x70` dla hydroboksu).
* **`DST`**: Adres odbiorcy (`0x70` dla hydroboksu, `0x60` dla pilota).
* **`CHECKSUM`**: 8-bitowa suma kontrolna modulo 256 wszystkich bajtów od `LEN` do bajtu poprzedzającego sumę:
  $$\text{CHECKSUM} = \left(\sum_{i=0}^{N-1} \text{bajt}[i]\right) \pmod{256}$$

---

## 4. Temperatura Wody Zasilania Strefy 1 (Bajt 13)

> [!IMPORTANT]
> **Błąd w projektach społeczności:** Poprzednie implementacje czytały temperaturę wody zasilania strefy 1 z `raw[14]`, co dawało stałą wartość `-16.0 °C`.  
> **Poprawny bajt:** Rzeczywista temperatura wody zasilania ($T_{WO}$) znajduje się w **bajcie 13** (`raw[13]`):
> $$T_{WO}\ [^\circ\text{C}] = \frac{\text{raw}[13]}{2} - 16$$

---

## 5. Ramki Sterujące (Pilot $\to$ Hydrobox)

### 5.1 Zasilanie Strefy 1 C.O. (`0x21`)
* **WYŁĄCZ (OFF):**
  ```text
  0B:60:70:E0:01:21:02:DF
  ```
* **WŁĄCZ (HEAT):**
  ```text
  0B:60:70:E0:01:21:03:E0
  ```

### 5.2 Tryb Fabrycznej Krzywej Pogodowej AUTO (`0x24:01`)
* **STAŁA NASTAWA (MANUAL):**
  ```text
  0C:60:70:E0:01:24:01:00:E2
  ```
* **KRZYWA POGODOWA AUTO:**
  ```text
  0C:60:70:E0:01:24:01:01:E3
  ```

### 5.3 Nastawa Zadanej Temperatury Wody (`0x23:02`)
Wysyła docelową temperaturę zasilania ($T_{cel} \in [20, 55]^\circ\text{C}$):
```text
0F:60:70:E0:01:23:02:<ENC>:66:A2:<ENC>:<CHK>
```
Gdzie:
$$\text{ENC} = (T_{cel} + 16) \times 2$$
$$\text{CHK} = \text{SumaKontrolna}(0x0F, \dots, \text{ENC})$$

### 5.4 Tryb Cichy Agregatu / Silent Mode (`0x24:04`)
* **TRYB CICHY WYŁĄCZONY:**
  ```text
  0C:60:70:E0:01:24:04:00:E5
  ```
* **TRYB CICHY WŁĄCZONY:**
  ```text
  0C:60:70:E0:01:24:04:04:E9
  ```

---

## 6. Protokół Zdalnego Odczytu i Zapisu Kodów Instalatora (Kody DN)

Uniwersalny protokół pozwalający odpytywać i modyfikować parametry EEPROM pompy bezpośrednio przez linię AB, bez wchodzenia w menu instalatora na pilocie ściennym:

### 6.1 Ramka Zapisu Kodu DN (`0x1C`):
Zapis nastawy bezpośrednio do pamięci EEPROM:
```text
F0:F0:0C:60:70:E0:01:1C:<DN_HEX>:<DATA_HEX>:<CHK>:A0
```
Odpowiedź potwierdzenia hydroboxu (Master ACK):
```text
0A:70:60:E0:80:A1:DB
```

*Przykład przechwycony na żywo (zapis przesunięcia krzywej DN 27 = +2 K):*
```text
Pilot TX:    0C:60:70:E0:01:1C:27:02:02
Hydrobox RX: 0A:70:60:E0:80:A1:DB (ACK)
```

### 6.2 Ramka Odpytania Kodu DN (`0x1A:F1`):
Zapytanie pilota:
```text
0D:60:70:E0:41:1A:F1:<DN_HEX>:00:<CHK>
```
Odpowiedź hydroboxu (`0x1A`):
```text
0C:70:60:E0:80:1A:<DN_HEX>:<VAL_HEX>:<CHK>
```

### 6.3 Ramki Wyjścia z Menu Instalatora:
```text
0A:60:70:E0:00:62:1C
0A:60:FF:E0:88:62:33
```
