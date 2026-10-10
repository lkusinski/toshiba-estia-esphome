# Kody Instalatora DN — Pompy Ciepła Toshiba Estia

> **Język / Language:** [🇬🇧 English](DN-CODES-REFERENCE.md) | [🇵🇱 Polski](DN-CODES-REFERENCE.pl.md)

Zestawienie kluczowych parametrów instalatora (kody DN) dla pomp Toshiba Estia (seria 4 i 5), które można odczytywać i modyfikować przez magistralę AB.

---

## 1. Parametry Krzywej Grzewczej i Ogrzewania

| DN (Hex) | DN (Dec) | Nazwa Parametru | Domyślnie | Zakres | Jednostka | Opis |
|---|---|---|---|---|---|---|
| `0x27` | **DN 27** | Przesunięcie krzywej grzewczej (Offset) | `0x00` (0) | -5 do +5 | K / °C | Równoległe przesunięcie pionowe krzywej grzania wody. |
| `0x28` | **DN 28** | Temp. wody zasilania przy ciepłej pogodzie $T_{W1}$ | `0x19` (25) | 20 do 55 | °C | Docelowa woda C.O. przy temperaturze zewnętrznej TE1. |
| `0x29` | **DN 29** | Temp. wody zasilania przy mrozie $T_{W2}$ | `0x2A` (42) | 25 do 55 | °C | Docelowa woda C.O. przy temperaturze zewnętrznej TE2. |
| `0x2A` | **DN 30** | Graniczna temp. zewnętrzna (Ciepło) $T_{E1}$ | `0x11` (17) | 10 do 25 | °C | Próg wysokiej temperatury zewnętrznej dla krzywej. |
| `0x2B` | **DN 31** | Graniczna temp. zewnętrzna (Mróz) $T_{E2}$ | `0xF1` (-15)| -20 do 5 | °C | Projektowa temperatura mrozu dla krzywej grzewczej. |

---

## 2. Histereza Termostatu i Pompa Obiegowa

| DN (Hex) | DN (Dec) | Nazwa Parametru | Domyślnie | Zakres | Jednostka | Opis |
|---|---|---|---|---|---|---|
| `0x19` | **DN 25** | Histereza załączenia termostatu C.O. | `0x03` (3) | 1 do 10 | K | Spadek temperatury poniżej zadanej przed startem sprężarki. |
| `0x1A` | **DN 26** | Histereza wyłączenia termostatu C.O. | `0x05` (5) | 1 do 10 | K | Przebicie temperatury powyżej zadanej przed wyłączeniem sprężarki. |
| `0x24` | **DN 36** | Bieg pompy obiegowej Grundfos UPM2 | `0x06` (6) | 1 do 7 | Bieg | Profil wydajności pompy obiegowej wewnątrz hydroboksu. |
| `0x27` | **DN 39** | Czas wybiegu pompy obiegowej | `0x08` (8) | 1 do 30 | min | Czas pracy pompy po wyłączeniu termostatycznym sprężarki. |

---

## 3. Limity Mocy i Grzałka Szczytowa

| DN (Hex) | DN (Dec) | Nazwa Parametru | Domyślnie | Zakres | Jednostka | Opis |
|---|---|---|---|---|---|---|
| `0x44` | **DN 68** | Limit prądu agregatu zewnętrznego | `0x0F` (15) | 5 do 20 | A | Maksymalny prąd pobierany przez inwerter sprężarki. |
| `0x09` | **DN 09** | Zwłoka załączenia grzałki szczytowej 6 kW | `0x4B` (75) | 10 do 120| min | Czas opóźnienia przed dołączeniem grzałki elektrycznej. |

---

## 4. Sterowanie Kodami DN z Home Assistant

Oprogramowanie ESPHome udostępnia:
* Suwak w UI: `number.toshiba_estia_r410a_estia_przesuniecie_krzywej_dn_27`.
* Serwisy API ESPHome:
  * `esphome.toshiba_estia_r410a_write_dn_code(code: 39, value: 2)`
  * `esphome.toshiba_estia_r410a_query_dn_code(code: 39)`
