# ❄️ Toshiba Estia — Integracja ESPHome przez Magistralę AB (AW-LINK)

> **Język / Language:** [🇬🇧 English](README.md) | [🇵🇱 Polski](README.pl.md)

[![Wersja ESPHome](https://img.shields.io/badge/ESPHome-2026.8%2B-teal.svg)](https://esphome.io/)
[![Sprzęt](https://img.shields.io/badge/Hardware-Makusets%20v3.2%20(ESP--12F)-orange.svg)](docs/HARDWARE-GUIDE.pl.md)
[![Licencja: MIT](https://img.shields.io/badge/Licencja-MIT-blue.svg)](LICENSE)
[![Protokół](https://img.shields.io/badge/Protokół-AW--LINK%20(2400%208N1)-purple.svg)](docs/PROTOCOL-AW-LINK.pl.md)
[![GitHub Fork](https://img.shields.io/badge/Fork-makusets%2Fesphome--toshiba--ab-green.svg)](https://github.com/lkusinski/esphome-toshiba-ab)

> [!NOTE]
> **Kompletne rozwiązanie i oficjalny wkład w Upstream:**
> Projekt stanowi gotowy pakiet integracyjny dla pomp Toshiba Estia serii 4 i 5 Hipower. Nasze poprawki i ulepszenia zostały oficjalnie połączone z nadrzędnym repozytorium makusets przez [**Pull Request #245**](https://github.com/makusets/esphome-toshiba-ab/pull/245) (rozwiązując [Issue #242](https://github.com/makusets/esphome-toshiba-ab/issues/242)).

Kompletne, przetestowane na żywym sprzęcie rozwiązanie integrujące pompy ciepła **Toshiba Estia (seria 4 oraz 5 Hipower, R410A / R32)** z **Home Assistant** przez 2-żyłową, niespolaryzowaną **magistralę AB (AW-LINK)**.

Projekt zawiera:
1. Autonomiczny firmware **ESPHome w C++** z natywnym termostatem strefy C.O. o rozdzielczości 0,5 K.
2. Sprzętowy limit modulacji **Silent Mode (30 Hz)** eliminujący taktowanie przy rozruchu.
3. Zestawienie kodów instalatora (DN) z ramkami zdalnego zapisu i odczytu.
4. Gotowy pakiet automatyzacji Home Assistant (autoadaptacja, Room Cut-off, optymalizacja PV, taryfa G12w, tryb letni i audyt fizycznego podlicznika).
5. Zaawansowany widok dashboardu Lovelace z wykresami modulacji sprężarki, ciśnień i COP.

### 🛠️ Zweryfikowano na instalacji fizycznej:
* **Jednostka wewnętrzna (Hydrobox):** `HWS-P805XWHT6-E` (Estia 5 Hipower)
* **Agregat zewnętrzny:** `HWS-P805HR-E` (8 kW, czynnik R410A)
* **Pilot ścienny:** `HWS-AMS54E`
* **Transceiver:** Płytka Makusets v3.2 (zasilana bezpośrednio z magistrali AB hydroboksu)

---

## ⚙️ Funkcje i uwagi techniczne

1. **Natywny termostat C++ w ESPHome z rozdzielczością 0,5 K (`climate`):**
   * Brak konieczności stosowania sztucznych szablonów `climate_template` czy automatyzacji synchronizujących tryby.
   * Bezpośrednie sterowanie trybami pompy (`OFF`, `HEAT`, `AUTO` krzywa pogodowa) oraz nastawą temperatury zasilania (20–55 °C) za pomocą natywnych ramek (`0x21`, `0x24:01`, `0x23:02`).
   * **Natywna obsługa kroków co 0,5°C:** Magistrala AW-LINK natywnie przyjmuje nastawy połówkowe, mimo że fizyczny pilot ścienny pozwala na skoki tylko o 1°C! Encja w HA wystawia `target_temp_step: 0.5`, co pozwala precyzyjnie wystroić bezpieczną podłogę krzywej (np. 31,5°C).
2. **Sprzętowy limit modulacji inwertera (Silent Mode 30 Hz):**
   * Wyeliminowano problem ucinania cykli grzania w ciepłe dni: fabryczny agresywny rozruch 42–48 Hz przegrzewał mały zład wody w kilkanaście minut.
   * Narzucenie sprzętowego sufitu **30,0 Hz** (ramka Silent Mode `0x24:04`) umożliwia łagodne wejście w modulację i zejście do **10–11 Hz**. Wydłużyło to cykle grzania do ciągłych **30+ minut** i podniosło dobowy **COP do 7,68 – 7,72**.
3. **Poprawka dekodowania temperatury wody strefy 1 (Fix bajtu 13):**
   * Rozwiązano problem stałego odczytu `-16.0 °C` występujący we wcześniejszych projektach społeczności (które błędnie czytały bajt 14). Rzeczywista temperatura wody zasilania $T_{WO}$ znajduje się w **bajcie 13** ($T = \text{raw}[13]/2 - 16$).
4. **Zdalny odczyt i zapis kodów instalatora EEPROM (Kody DN):**
   * Zdekodowano uniwersalny protokół zapisu kodów instalatora **0x1C** oraz zapytań **0x1A:F1**.
   * Możliwość zdalnej zmiany histerezy termostatu (DN 25/26), biegu pompy obiegowej Grundfos (DN 36), limitu prądu agregatu (DN 68) oraz przesunięcia krzywej grzewczej (DN 27) bez dotykania pilota ściennego.
5. **Pełna telemetria obiegu chłodniczego i czujnik pilota ściennego:**
   * Ciśnienie wysokie ($P_D$) i niskie ($P_S$) w kPa.
   * Temperatura tłoczenia ($T_D$), ssania ($T_S$), wymiennika ($T_E$), inwertera ($T_{HS}$) i skroplin ($T_C$).
   * Temperatura otoczenia ze ściennego pilota w garażu dekodowana wprost z ramek pingu (`0x81`).
   * Częstotliwość pracy sprężarki (Hz), kroki elektronicznego zaworu rozprężnego (PMV), obroty wentylatora (RPM).
6. **Liczniki energii i COP bezpośrednio na mikrokontrolerze:**
   * Obliczanie mocy cieplnej i elektrycznej w czasie rzeczywistym.
   * Dobowe liczniki wyprodukowanego ciepła (kWh), pobranego prądu (kWh) oraz średniego dobowego COP liczone na procesorze ESP, co eliminuje obciążenie bazy danych Home Assistanta.
7. **Stabilne zasilanie z magistrali AB i bezpieczne aktualizacje OTA:**
   * Wyeliminowano problem restartów płytki podczas bezprzewodowego wgrywania oprogramowania (OTA) poprzez optymalizację mocy Wi-Fi (`output_power: 14.0dBm`).
   * Wizualna sygnalizacja stanu magistrali i połączenia niebieską diodą LED na **GPIO2**.
   * Sprzętowy hebel bezpieczeństwa (`read_only_switch: ALWAYS_ON`) odcina nadawanie na magistralę na poziomie procesora.
8. **Autoadaptacja i Termostat Pokojowy (Room Cut-off):**
   * Bezpośrednie zadawanie pożądanej temperatury w domu (`input_number.estia_oczekiwana_temperatura_w_domu`, np. 21.0°C) – algorytm sam dynamicznie koryguje temperaturę wody zasilającej ze sprzężeniem Kp.
   * **Podwójna histereza Schmitta Room Cut-off:** Gdy temperatura w domu przekroczy cel o próg wyłączenia (>= +0,2 K), obieg grzewczy jest całkowicie wyłączany (`hvac_mode: off`), zatrzymując sprężarkę i pompę obiegową w okresach przejściowych. Automatyczne wznowienie (`hvac_mode: heat`) następuje po schłodzeniu budynku (<= -0,1 K).
9. **Kalibracja i audyt z fizycznym podlicznikiem prądu:**
   * Gotowe encje pomocnicze i sensory w HA (`input_number.estia_licznik_fizyczny_stan`) do porównywania wirtualnego zliczania energii z szyny hydroboksu z fizycznym podlicznikiem w rozdzielnicy.
   * Bieżące wyliczanie odchyłki rzeczywistej i współczynnika korygującego $k = \frac{\Delta E_{\text{fiz}}}{\Delta E_{\text{HA}}}$ (weryfikacja poboru własnego pompy Grundfos, płyty głównej i grzałki karteru).
10. **Nadzór operacyjny, ochrona nastaw i audyt cykliczności:**
   * **Koszyk FIFO 5 ostatnich akcji:** rozróżnianie ingerencji domownika na pilocie ściennym od korekt automatycznych, ochrona nastawy ręcznej (`Manual Override`) oraz przycisk natychmiastowego powrotu do automatyki pogodowej.
   * **Monitoring taktowania sprężarki:** sensory `history_stats` zliczające cykle pracy w oknach 1h, 24h i od północy.

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
