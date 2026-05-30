# 🪟 Dachluke Badentlüftung

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml)

Intelligente Steuerung einer motorisierten Dachluke (**Homematic HmIP-BROLL**) zur Badentlüftung – vollständig konfigurierbar über die Home Assistant Web-UI.

---

## ✨ Features

- **Hygrometer-gesteuert** – öffnet bei hoher Luftfeuchtigkeit
- **Taupunkt-basiertes Schließen** – physikalisch korrekt via Thermal Comfort + Met.no (optional)
- **Lichtsperre** – öffnet nur wenn Badezimmerlampe an ist
- **Drei Wetterkategorien** – normal / eingeschränkt / gesperrt
- **Gesperrtes Wetter** – Auto-Öffnen verboten; manuelles Öffnen auf Minimalposition mit Alexa-Warnung
- **DWD-Warnstufe** – optional in Alexa-Ansage bei manuellem Öffnen trotz gesperrtem Wetter
- **Zuverlässige Positionsüberwachung** – Kanal 3 des HmIP-BROLL
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
| Licht | Mind. eine `light`-Entität |
| Wetterintegration | `weather`-Entität mit `dew_point`-Attribut (z. B. Met.no) |
| Thermal Comfort (optional) | HACS – für taupunkt-basiertes Schließen |
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

## 🔧 Helfer anlegen

| Typ | Entity-ID | Zweck |
|---|---|---|
| Toggle | `input_boolean.dachluke_pause` | Temporäre Pause |
| Toggle | `input_boolean.dachluke_deaktiviert` | Dauerhafte Deaktivierung |
| Timer | `timer.dachluke_lueftung` | Maximale Öffnungsdauer |
| Text | `input_text.dachluke_status` | Dashboard-Statusanzeige |

---

## 🎮 Bedienung

| Taste | Kurzer Druck | Langer Druck |
|---|---|---|
| **Öffnen** | Timer startet | Automation aktivieren 🟢 |
| **Schließen** | Pause | Automation deaktivieren 🔴 |

---

## 💧 Schließlogik

**Standard (relative Feuchte):**
```
Schließen wenn rF < Öffnungsschwellwert − Hysterese
Beispiel: 75% − 10% = 65%
```

**Optional (Taupunkt-basiert via Thermal Comfort):**
```
Schließen wenn Innen-Taupunkt < Außen-Taupunkt + Offset
Beispiel: 12.5°C + 2°C = 14.5°C
```
Taupunkt hat Vorrang wenn verfügbar, rF ist immer Fallback.

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
  → rF > 75% + Licht an + nicht gesperrt
  → Luke öffnet + Timer startet

Lüftung läuft...
  → rF < 65% oder Taupunkt-Bedingung   → schließen ✅
  → Timer abgelaufen (45 min)           → Zwangsschließung ⏱️
  → Wetter wechselt zu gesperrt         → sofort schließen 🌪️
  → Alarm aktiviert                     → sofort schließen 🚨
  → Kurzer Druck Schließen              → Pause ⏸️
  → Langer Druck Schließen              → deaktiviert 🔴
  → Langer Druck Öffnen                 → aktiv 🟢
  → Positionssensor offline             → kein Auto-Öffnen,
                                           Timer/Alarm schließen trotzdem
```

---

## 📝 Changelog

### v0.6.9 (aktuell)
Produktionsreife Version nach ausführlichem Code-Review.

Neue Features: Taupunkt-Schließen, drei Wetterkategorien, gesperrtes Wetter mit manuellem Override, DWD-Integration, robuste Positionssensor-Behandlung, Wetterkorrektur-Meldungen mit alt→neu.

Bugfixes: Timer-Cancel bei manuellem Schließen, Race-Condition Positionssensor, position_changed-Fallback, Alarm ohne is_open, Jinja-Syntax, HA 2026.x Syntax.

### v0.5.1
Erste stabile Version: Event-Trigger, getrennte Helfer für Pause/Deaktivierung, kein blockierendes delay.

---

## 👤 Autor

**C4mp3r-Grey** – [github.com/C4mp3r-Grey](https://github.com/C4mp3r-Grey)
