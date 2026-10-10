# Przewodnik Sprzętowy — Płytka Makusets v3.2 i Magistrala AB

> **Język / Language:** [🇬🇧 English](HARDWARE-GUIDE.md) | [🇵🇱 Polski](HARDWARE-GUIDE.pl.md)

Instrukcja dotycząca fizycznego podłączenia, zworki zasilania, programowania oraz eliminacji zakłóceń modułu ESP-12F (ESP8266) na magistrali Toshiba AB.

---

## 1. Budowa Płytki Transceivera Makusets v3.2

Dedykowana płytka **Makusets v3.2** została zaprojektowana specjalnie do komunikacji z magistralami klimatyzacji i pomp ciepła Toshiba / Carrier AB:
* **Prostownik wejściowy:** Mostek Graetza `MB10S` eliminuje konieczność zachowywania polaryzacji na przewodach A i B.
* **Zasilacz impulsowy:** Wysokosprawna przetwornica step-down `K7803-500` obniża napięcie magistrali (~15–18 V DC) do stabilnego 3.3 V DC.
* **Mikrokontroler:** Moduł ESP-12F (ESP8266) z niebieską diodą LED podłączoną do pinu **GPIO2**.
* **Piny magistrali UART:**
  * **TX:** `GPIO12` (nadawanie na magistralę przez układ tranzystorowy)
  * **RX:** `GPIO13` (odbiór z magistrali przez komparator)
* **Port USB:** Układ CH340 USB-UART do wstępnego wgrania oprogramowania z komputera.

---

## 2. Zworka Wyboru Zasilania (`U5`)

> [!CAUTION]
> **NIGDY nie podłączaj jednocześnie kabla USB i zasilania z magistrali AB przy niewłaściwej pozycji zworki!**

* **Zworka w pozycji `USB`:** Moduł zasilany jest wyłącznie z portu USB 5V (przez stabilizator AMS1117). Stosuj tę pozycję na biurku podczas pierwszego programowania przez kabel.
* **Zworka w pozycji `AB`:** Moduł zasilany jest bezpośrednio z magistrali hydroboksu. Przed przestawieniem zworki na `AB` bezwzględnie **odłącz kabel USB**.

---

## 3. Kluczowe Wskazówki Sprzętowe

### 3.1 Pierwsze Programowanie przez USB (Specyfika CH340)
Układ CH340 na płytkach Makusets v3.2 nie posiada podłączonych linii DTR/RTS do pinów GPIO0 oraz RST mikrokontrolera.
* **Procedura wejścia w tryb bootloadera:**
  1. Wciśnij i przytrzymaj przycisk **BOOT**.
  2. Wciśnij i puść przycisk **RESET**.
  3. Puść przycisk **BOOT**.
  4. Uruchom programowanie (np. `esphome run toshiba-estia-r410a.yaml`).
* **Po zakończeniu programowania:** Należy **fizycznie kliknąć RESET** na płytce, ponieważ automatyczny restart z poziomu komputera nie zadziała sprzętowo.

### 3.2 Zabezpieczenie przed Restartem (Brownout) przy Aktualizacjach OTA na Linii AB
Podczas zasilania z linii AB przez przetwornicę `K7803-500`, domyślna maksymalna moc nadajnika Wi-Fi (20 dBm, ok. 200 mA w impulsie) w trakcie kasowania pamięci flash (`erase_flash`) powoduje chwilowy spadek napięcia 3.3 V, co skutkuje resetem procesora i przerwaniem aktualizacji OTA.

**Rozwiązanie:**  
Ograniczenie mocy nadajnika Wi-Fi w konfiguracji ESPHome:
```yaml
wifi:
  output_power: 14.0dBm
```
Zmniejsza to pobór prądu w szczycie o ok. 100 mA i zapewnia 100% stabilne aktualizacje bezprzewodowe OTA bezpośrednio w kotłowni.

---

## 4. Podłączenie do Płyty Głównej Hydroboksu

Do połączenia wystarczy 2-żyłowy przewód (np. kabel alarmowy lub skrętka komputerowa):
* Zaciski na płycie głównej hydroboksu: **`CN02` lub `CN03` (oznaczone A i B)**.
* Moduł podłącza się **równolegle** z istniejącym pilotem przewodowym `HWS-AMS54E`.
* Połączenie jest niespolaryzowane (kolejność przewodów A i B jest dowolna).
