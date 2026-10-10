# ❄️ Toshiba Estia — Integracja ESPHome przez Magistralę AB (AW-LINK)

> **Język / Language:** [🇬🇧 English](README.md) | [🇵🇱 Polski](README.pl.md)

[![Wersja ESPHome](https://img.shields.io/badge/ESPHome-2026.8%2B-teal.svg)](https://esphome.io/)
[![Sprzęt](https://img.shields.io/badge/Hardware-Makusets%20v3.2%20(ESP--12F)-orange.svg)](docs/HARDWARE-GUIDE.pl.md)
[![Licencja: MIT](https://img.shields.io/badge/Licencja-MIT-blue.svg)](LICENSE)
[![Protokół](https://img.shields.io/badge/Protokół-AW--LINK%20(2400%208N1)-purple.svg)](docs/PROTOCOL-AW-LINK.pl.md)
[![GitHub Fork](https://img.shields.io/badge/Fork-makusets%2Fesphome--toshiba--ab-green.svg)](https://github.com/lkusinski/esphome-toshiba-ab)

Kompletne, przetestowane na żywym sprzęcie rozwiązanie integrujące pompy ciepła **Toshiba Estia (seria 4 oraz 5 Hipower, R410A / R32)** z **Home Assistant** przez 2-żyłową, niespolaryzowaną **magistralę AB (AW-LINK)**.

Projekt zawiera:
1. Autonomiczny firmware **ESPHome w C++** z natywnym termostatem strefy C.O.
2. Zestawienie kodów instalatora (DN) z ramkami zdalnego zapisu i odczytu.
3. Gotowy pakiet automatyzacji Home Assistant (optymalizacja PV, taryfa G12w, tryb letni).
4. Zaawansowany widok dashboardu Lovelace z wykresami modulacji sprężarki, ciśnień i COP.

### 🛠️ Zweryfikowano na instalacji fizycznej:
* **Jednostka wewnętrzna (Hydrobox):** `HWS-P805XWHT6-E` (Estia 5 Hipower)
* **Agregat zewnętrzny:** `HWS-P805HR-E` (8 kW, czynnik R410A)
* **Pilot ścienny:** `HWS-AMS54E`
* **Transceiver:** Płytka Makusets v3.2 (zasilana bezpośrednio z magistrali AB hydroboksu)

---

## ⚙️ Funkcje i uwagi techniczne

1. **Natywny termostat C++ w ESPHome (`climate`):**
   * Brak konieczności stosowania sztucznych szablonów `climate_template` czy automatyzacji synchronizujących tryby.
   * Bezpośrednie sterowanie trybami pompy (`OFF`, `HEAT`, `AUTO` krzywa pogodowa) oraz nastawą temperatury zasilania (20–55 °C) za pomocą natywnych ramek (`0x21`, `0x24:01`, `0x23:02`).
2. **Poprawka dekodowania temperatury wody strefy 1 (Fix bajtu 13):**
   * Rozwiązano problem stałego odczytu `-16.0 °C` występujący we wcześniejszych projektach społeczności (które błędnie czytały bajt 14). Rzeczywista temperatura wody zasilania $T_{WO}$ znajduje się w **bajcie 13** ($T = \text{raw}[13]/2 - 16$).
3. **Zdalny odczyt i zapis kodów instalatora EEPROM (Kody DN):**
   * Zdekodowano uniwersalny protokół zapisu kodów instalatora **0x1C** oraz zapytań **0x1A:F1**.
   * Możliwość zdalnej zmiany histerezy termostatu (DN 25/26), biegu pompy obiegowej Grundfos (DN 36), limitu prądu agregatu (DN 68) oraz przesunięcia krzywej grzewczej (DN 27) bez dotykania pilota ściennego.
4. **Pełna telemetria obiegu chłodniczego i freonu R410A:**
   * Ciśnienie wysokie ($P_D$) i niskie ($P_S$) w kPa.
   * Temperatura tłoczenia ($T_D$), ssania ($T_S$), wymiennika ($T_E$), inwertera ($T_{HS}$) i skroplin ($T_C$).
   * Częstotliwość pracy sprężarki (Hz), kroki elektronicznego zaworu rozprężnego (PMV), obroty wentylatora (RPM).
5. **Liczniki energii i COP bezpośrednio na mikrokontrolerze:**
   * Obliczanie mocy cieplnej i elektrycznej w czasie rzeczywistym.
   * Dobowe liczniki wyprodukowanego ciepła (kWh), pobranego prądu (kWh) oraz średniego dobowego COP liczone na procesorze ESP, co eliminuje obciążenie bazy danych Home Assistanta.
6. **Stabilne zasilanie z magistrali AB i bezpieczne aktualizacje OTA:**
   * Wyeliminowano problem restartów płytki podczas bezprzewodowego wgrywania oprogramowania (OTA) poprzez optymalizację mocy Wi-Fi (`output_power: 14.0dBm`).
   * Wizualna sygnalizacja stanu magistrali i połączenia niebieską diodą LED na **GPIO2**.
   * Sprzętowy hebel bezpieczeństwa (`read_only_switch: ALWAYS_ON`) odcina nadawanie na magistralę na poziomie procesora.

---

## 📁 Struktura Repozytorium

```text
├── README.md                          # Dokumentacja w języku angielskim
├── README.pl.md                       # Dokumentacja w języku polskim
├── LICENSE                            # Licencja MIT
├── docs/
│   ├── PROTOCOL-AW-LINK.md            # Specyfikacja protokołu magistrali AB (EN)
│   ├── PROTOCOL-AW-LINK.pl.md         # Specyfikacja protokołu magistrali AB (PL)
│   ├── DN-CODES-REFERENCE.md          # Mapa kodów instalatora DN (EN)
│   ├── DN-CODES-REFERENCE.pl.md       # Mapa kodów instalatora DN (PL)
│   ├── HARDWARE-GUIDE.md              # Przewodnik po płytce Makusets v3.2 i zasilaniu (EN)
│   └── HARDWARE-GUIDE.pl.md           # Przewodnik po płytce Makusets v3.2 i zasilaniu (PL)
├── esphome/
│   ├── toshiba-estia-r410a.yaml       # Gotowy plik konfiguracyjny ESPHome
│   ├── secrets.yaml.example           # Szablon poświadczeń Wi-Fi i kluczy API
│   └── components/
│       └── toshiba_ab/                # Kod źródłowy komponentu C++ (lokalny)
│           ├── __init__.py
│           ├── climate.py
│           ├── automation.h
│           ├── toshiba_ab.cpp
│           └── toshiba_ab.h
└── home-assistant/
    ├── packages/
    │   └── toshiba_estia_optymalizator.yaml  # Gotowy pakiet HA (PV, G12w, tryb letni)
    └── dashboards/
        └── lovelace_pompa_ciepla.json        # Gotowy widok dashboardu Lovelace
```

---

## 🚀 Instrukcja Krok po Kroku (Quick Start)

### Krok 1: Przygotowanie sprzętu
1. Na płytce Makusets v3.2 ustaw zworkę zasilania `U5` w pozycji **`USB`**.
2. Podłącz płytkę do komputera kablem micro-USB (z liniami danych).
3. Wprowadź ESP w tryb programowania (bootloadera): przytrzymaj przycisk **BOOT**, kliknij i puść **RESET**, a następnie puść **BOOT**.

### Krok 2: Konfiguracja i wgranie ESPHome
1. Skopiuj `esphome/secrets.yaml.example` do `esphome/secrets.yaml` i uzupełnij dane swojej sieci Wi-Fi oraz klucz API.
2. Skompiluj i wgraj oprogramowanie:
   ```bash
   esphome run esphome/toshiba-estia-r410a.yaml
   ```
3. Po zakończeniu flashowania **wciśnij fizycznie przycisk RESET** na płytce (układ CH340 nie resetuje ESP automatycznie).

### Krok 3: Podłączenie do pompy ciepła
1. Odłącz kabel USB od płytki.
2. Przestaw zworkę zasilania `U5` w pozycję **`AB`**.
3. **Wyłącz bezpiecznik zasilania pompy ciepła** przed podłączeniem przewodów.
4. Podłącz zaciski **A** i **B** płytki do złącz `CN02` lub `CN03` na płycie głównej hydroboksu (równolegle z istniejącym pilotem ściennym; brak polaryzacji).
5. Załącz zasilanie pompy. Moduł uruchomi się w ciągu kilku sekund, a niebieska dioda LED na module ESP-12F zasygnalizuje stabilną pracę.

### Krok 4: Wdrożenie w Home Assistant
1. Home Assistant automatycznie wykryje urządzenie przez integrację ESPHome.
2. Skopiuj plik `home-assistant/packages/toshiba_estia_optymalizator.yaml` do katalogu `/config/packages/` w swoim Home Assistancie.
3. Zaimportuj plik `home-assistant/dashboards/lovelace_pompa_ciepla.json` jako nowy widok w dashboardzie Lovelace.

---

## 📜 Uznanie autorstwa (Credits)
* Bazuje na pracach inżynierii wstecznej prowadzonych przez [makusets/esphome-toshiba-ab](https://github.com/makusets/esphome-toshiba-ab), pionierskich badaniach warstwy fizycznej transceivera przez [issalig/toshiba_air_cond](https://github.com/issalig/toshiba_air_cond) oraz implementacji TCC-Link w ESPHome autorstwa [muxa/esphome-tcc-link](https://github.com/muxa/esphome-tcc-link).
* Przetestowany, dostosowany i udokumentowany na domowej instalacji pompy ciepła Toshiba Estia 5 Hipower.  
* Oficjalny fork komponentu z poprawkami: [lkusinski/esphome-toshiba-ab](https://github.com/lkusinski/esphome-toshiba-ab).
