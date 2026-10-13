# 🪟 Dachluke Badentlüftung

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml)

Intelligente Steuerung einer motorisierten Dachluke (**Homematic HmIP-BROLL**) zur Badentlüftung – vollständig konfigurierbar über die Home Assistant Web-UI.

---
## ✨ Features
- **Feuchte-Steigungs-gesteuert** – öffnet bei schnellem Feuchteanstieg, z. B. beim Duschen
- **rF-basiertes Schließen** – schließt bei konfigurierbarem Feuchte-Schwellwert
- **Taupunkt-basiertes Schließen** – physikalisch korrekt via Thermal Comfort + Met.no (optional)
- **Zweites Nachlüften** – optionaler kurzer Nachlüft-Zyklus bei Restfeuchte
- **Frostmodus** – kleinere Öffnung und kürzere Lüftungsdauer bei Frost
- **Heizungssteuerung** – optionales Absenken beim Lüften und Rückkehr zur Zieltemperatur
- **Lichtsperre** – öffnet nur wenn Badezimmerlampe an ist
- **Drei Wetterkategorien** – normal / eingeschränkt / gesperrt
- **Gesperrtes Wetter** – Auto-Öffnen verboten; manuelles Öffnen auf Minimalposition mit Alexa-Warnung
- **DWD-Warnstufe** – optional in Alexa-Ansage bei manuellem Öffnen trotz gesperrtem Wetter
- **Zuverlässige Positionsüberwachung** – Kanal 3 des HmIP-BROLL
- **Fensterkontakt-Plausibilität** – optionaler Abgleich mit Wartezeit gegen Fehlalarme durch Sensor-Latenz
- **Kein doppelter Cover-Befehl** – manuelle Bedienung wird als Event erkannt
- **Kurzer Druck Öffnen** – Timer startet
- **Langer Druck Öffnen** – Automation aktivieren 🟢
- **Kurzer Druck Schließen** – Pause
- **Langer Druck Schließen** – Automation dauerhaft deaktivieren 🔴
- **Positionssensor-Ausfall** – sicherheitsorientiert behandelt
- **3 Benachrichtigungskanäle** – App, Dashboard, Alexa (SSML, Lautstärke wählbar)
---
## 📋 Voraussetzungen
| Voraussetzung | Details |
|---|---|
| Home Assistant | mind. 2025.6.0 |
| Homematic(IP) Local | HACS, OpenCCU oder RaspberryMatic |
| Cover-Entität | Kanal 4 des HmIP-BROLL |
| Positionssensor | Kanal 3 – manuell aktivieren (siehe unten) |
| Hygrometer | Sensor mit `device_class: humidity` |
| Feuchte-Steigungssensor | Statistics-Sensor mit `state_characteristic: change` über ca. 5 Minuten |
| Licht | Mind. eine `light`-Entität |
| Wetterintegration | `weather`-Entität, idealerweise mit `dew_point`-Attribut (z. B. Met.no) |
| Thermal Comfort (optional) | HACS – für taupunkt-basiertes Schließen |
| Climate/Thermostat (optional) | Für automatische Heizungsabsenkung |
| Zusätzliche Timer (optional) | Für zweites Nachlüften |
| DWD Warnungen (optional) | HA-Integration – für Warnstufe in Alexa-Ansage |
| Alexa (optional) | Alexa Devices Integration (HA 2025.6+) |
| Alarmo (optional) | Alarmo Integration |
---
## 🔓 Kanal 3 aktivieren

1. **Einstellungen → Geräte & Dienste → Homematic(IP) Local**
2. HmIP-BROLL Gerät öffnen
3. **Unignore Parameters** → Kanal 3 → `LEVEL` aktivieren
4. HA neu starten → `sensor.dachluke_ch3` erscheint

---
## 📈 Feuchte-Steigungssensor anlegen

Der Blueprint öffnet nicht mehr nur wegen absolut hoher Luftfeuchtigkeit, sondern wegen eines schnellen Feuchteanstiegs. Dafür wird ein Statistics-Sensor benötigt, der die Änderung des Hygrometers über ein Zeitfenster berechnet.

Beispiel in `configuration.yaml` oder in einer ausgelagerten Sensor-Datei:

```yaml
sensor:
  - platform: statistics
    name: Bad Feuchte Steigung 5min
    entity_id: sensor.bad_luftfeuchtigkeit
    state_characteristic: change
    max_age:
      minutes: 5
```

Diesen Sensor anschließend im Blueprint als **Feuchte-Steigungssensor** auswählen. Der Standardschwellwert ist `8`, also z. B. ein Anstieg von 55% auf 63% innerhalb von ca. 5 Minuten.

---
## 🔧 Helfer anlegen

| Typ | Entity-ID | Zweck |
|---|---|---|
| Toggle | `input_boolean.dachluke_pause` | Temporäre Pause |
| Toggle | `input_boolean.dachluke_deaktiviert` | Dauerhafte Deaktivierung |
| Timer | `timer.dachluke_lueftung` | Maximale Öffnungsdauer |
| Text | `input_text.dachluke_status` | Dashboard-Statusanzeige |
| Timer (optional) | `timer.dachluke_nachlueften_wartezeit` | Wartezeit vor zweitem Nachlüften |
| Timer (optional) | `timer.dachluke_nachlueften_dauer` | Dauer des zweiten Nachlüftens |

Die beiden Nachlüft-Timer werden nur benötigt, wenn **Zweites Nachlüften** aktiviert wird.

---
## 🎮 Bedienung

| Taste | Kurzer Druck | Langer Druck |
|---|---|---|
| **Öffnen** | Timer startet | Automation aktivieren 🟢 |
| **Schließen** | Pause | Automation deaktivieren 🔴 |

---
## 💧 Schließlogik

**Öffnen:**
```
Öffnen wenn Feuchte-Steigung > Schwellwert
und Licht an
und Wetter nicht gesperrt
und Automation nicht pausiert/deaktiviert
```

**Standard-Schließen (relative Feuchte):**
```
Schließen wenn rF < Schließ-Schwellwert
Beispiel: rF < 65%
```

**Optional (Taupunkt-basiert via Thermal Comfort):**
```
Schließen wenn Innen-Taupunkt < Außen-Taupunkt + Offset
Beispiel: 12.5°C < 12.5°C + 2°C
```
Taupunkt hat Vorrang wenn verfügbar, rF ist immer Fallback.

**Zweites Nachlüften (optional):**
```
Nach dem ersten Schließen warten
→ wenn Restfeuchte noch über Schwelle liegt
→ kurz erneut öffnen
```

---
## 🌧️ Wetterkategorien

| Kategorie | Verhalten |
|---|---|
| Normal | Volle Öffnung (Standard 80%) |
| Eingeschränkt | Minimale Öffnung (Standard 20%) |
| Gesperrt | Auto-Öffnen verboten, Wetterwechsel schließt sofort; manuelles Öffnen auf Minimalposition mit Alexa-Warnung |

---
## 🔄 Logik

```
Duschen beginnt
  → Feuchte steigt schnell + Licht an + nicht gesperrt
  → Luke öffnet + Timer startet
  → optional: Heizung wird abgesenkt
Lüftung läuft...
  → rF < Schließ-Schwellwert oder Taupunkt-Bedingung → schließen ✅
  → Timer abgelaufen (45 min)                        → Zwangsschließung ⏱️
  → optional: zweites Nachlüften bei Restfeuchte      → kurzer Nachlüft-Zyklus 🔄
  → Frost aktiv                                      → kleinere Öffnung, kürzerer Timer 🧊
  → Wetter wechselt zu gesperrt                      → sofort schließen 🌪️
  → Alarm aktiviert                                  → sofort schließen 🚨
  → Kurzer Druck Schließen                           → Pause ⏸️
  → Langer Druck Schließen                           → deaktiviert 🔴
  → Langer Druck Öffnen                              → aktiv 🟢
  → Positionssensor offline                          → kein Auto-Öffnen,
                                                        Timer/Alarm/Wetter schließen trotzdem
  → Positionssensor + Fensterkontakt widersprechen   → Meldung nach Stabilisierungszeit
```
---
## 📝 Changelog

### v0.7.5 (aktuell)
Produktionsnahe Version nach größerem Funktionsupdate und Review.

Neue Features: Feuchte-Steigungs-Trigger via Statistics-Sensor, fester rF-Schließwert, Heizungssteuerung, Frostmodus, zweites Nachlüften über eigene Timer, konfigurierbare Wartezeit vor Positions-/Kontaktvergleich.

Bugfixes: Zweites Nachlüften ohne blockierende `delay`s, frische Zustände nach Timer-Events, Nachlüft-Timer werden an Stop-/Sicherheitsstellen gecancelt, Hauptlüftung hat Vorrang vor Nachlüftung, Wetter-/Frost-Priorität korrigiert, Sensor-Widerspruch wird erst nach Stabilisierungszeit geprüft.

### v0.6.9
Produktionsreife Version nach ausführlichem Code-Review.

Neue Features: Taupunkt-Schließen, drei Wetterkategorien, gesperrtes Wetter mit manuellem Override, DWD-Integration, robuste Positionssensor-Behandlung, Wetterkorrektur-Meldungen mit alt→neu.

Bugfixes: Timer-Cancel bei manuellem Schließen, Race-Condition Positionssensor, position_changed-Fallback, Alarm ohne is_open, Jinja-Syntax, HA 2026.x Syntax.

### v0.5.1
Erste stabile Version: Event-Trigger, getrennte Helfer für Pause/Deaktivierung, kein blockierendes delay.

---

## 👤 Autor

**C4mp3r-Grey** – [github.com/C4mp3r-Grey](https://github.com/C4mp3r-Grey)
