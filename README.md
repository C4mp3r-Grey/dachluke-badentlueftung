# 🪟 Dachluke Badentlüftung

Home Assistant Blueprint zur intelligenten Steuerung einer motorisierten Dachluke zur Badentlüftung – vollständig konfigurierbar über die Home Assistant Web-UI, kein YAML-Editieren erforderlich.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml)

---

## ✨ Features

- **Hygrometer-gesteuert** – öffnet bei hoher Luftfeuchtigkeit, schließt wenn Luft trocken (mit konfigurierbarer Hysterese)
- **Lichtsperre** – öffnet nur wenn Badezimmerlampe an ist, kein Einfluss auf das Schließen
- **Wetterabhängige Position** – weit bei gutem Wetter, minimal bei Regen/Sturm
- **Wetterkorrektur** – passt Position bei Wetterwechsel an ohne den Schließtimer zurückzusetzen
- **Zuverlässige Positionsüberwachung** – über Kanal 3 des HmIP-BROLL statt unzuverlässigem Kanal 4
- **Zuverlässige Event-Trigger** – HmIP-BROLL Event-Entitäten statt Cover-State-Changes
- **Kurzer Druck Öffnen** – Luke öffnet manuell
- **Langer Druck Öffnen** – Automation aktivieren 🟢
- **Kurzer Druck Schließen** – Luke schließt + temporäre Pause
- **Langer Druck Schließen** – Automation dauerhaft deaktivieren 🔴
- **Zuverlässige Zwangsschließung** – nativer HA-Timer, schließt exakt bei 0
- **Alarmo-Integration** – sperrt bei aktivem Alarm, schließt sofort (optional)
- **Fensterkontakt** – optionale Plausibilitätsprüfung
- **3 Benachrichtigungskanäle** – App-Push, Dashboard-Popup, Alexa-Sprachausgabe (SSML, Lautstärke wählbar)

---

## 📋 Voraussetzungen

| Voraussetzung | Details |
|---|---|
| Home Assistant | mind. 2025.6.0 |
| Homematic(IP) Local | HACS-Integration, OpenCCU oder RaspberryMatic |
| Cover-Entität | Kanal 4 des HmIP-BROLL – nur für Steuerbefehle |
| Positionssensor | Kanal 3 des HmIP-BROLL – manuell aktivieren (siehe unten) |
| Event-Entitäten | Kanal 1 (Öffnen) und Kanal 2 (Schließen) des HmIP-BROLL |
| Hygrometer | Sensor mit `device_class: humidity` |
| Licht | Mind. eine `light`-Entität im Badezimmer |
| Wetterintegration | Beliebige `weather`-Entität |
| Alexa (optional) | Alexa Devices Integration (HA 2025.6+) |
| Alarmo (optional) | Alarmo Integration |

---

## 🔓 Kanal 3 aktivieren (Positionssensor)

Der HmIP-BROLL meldet die tatsächliche Position über Kanal 3 – dieser muss in der Homematic(IP) Local Integration manuell freigeschaltet werden:

1. **Einstellungen → Geräte & Dienste → Homematic(IP) Local**
2. HmIP-BROLL Gerät öffnen
3. **Unignore Parameters** → Kanal 3 → `LEVEL` aktivieren
4. HA neu starten → `sensor.dachluke_ch3` erscheint (Wert 0–100%)

---

## 🔧 Helfer anlegen

Drei Helfer unter **Einstellungen → Helfer → Erstellen** anlegen:

| Typ | Name | Entity-ID |
|---|---|---|
| Toggle | Dachluke Pause | `input_boolean.dachluke_pause` |
| **Timer** | Dachluke Lüftung | `timer.dachluke_lueftung` |
| Text | Dachluke Status | `input_text.dachluke_status` (Max. 255 Zeichen) |

> ⚠️ Seit v4.0 wird ein **Timer**-Helfer statt eines Datum/Uhrzeit-Helfers verwendet. Der alte `input_datetime.dachluke_schliessen_um` kann gelöscht werden.

---

## 🔍 Event-Entitäten finden

Unter **Entwicklerwerkzeuge → Ereignisse** auf `state_changed` lauschen und Taste drücken:

| Aktion | Entität (Beispiel) |
|---|---|
| Öffnen am Gerät | `event.dachluke_ch1` |
| Schließen am Gerät | `event.dachluke_ch2` |
| Öffnen Remote (WRC6) | `event.hmip_wrc6_bad_ch4` |
| Schließen Remote (WRC6) | `event.hmip_wrc6_bad_ch3` |

---

## 🎮 Bedienung

| Taste | Kurzer Druck | Langer Druck |
|---|---|---|
| **Öffnen** | Luke öffnet manuell | Automation aktivieren 🟢 |
| **Schließen** | Luke schließt + Pause ⏸️ | Automation deaktivieren 🔴 |

---

## 🚀 Installation

### 1. Blueprint importieren
Auf den Badge oben klicken oder diese URL manuell unter **Einstellungen → Automatisierungen & Szenen → Blueprints → Blueprint importieren** eingeben:
```
https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml
```

### 2. Kanal 3 aktivieren (siehe oben)

### 3. Drei Helfer anlegen (siehe oben)

### 4. Automation erstellen
**Einstellungen → Automatisierungen & Szenen → Blueprints → Dachluke Badentlüftung → Automatisierung erstellen**

---

## ⚙️ Konfiguration

### 🪟 Dachluke
| Parameter | Standard | Beschreibung |
|---|---|---|
| Cover-Entität | – | Steuerungsentität (Kanal 4) |
| Positionssensor | – | Kanal 3, z. B. `sensor.dachluke_ch3` |
| Position bei gutem Wetter | 80% | Öffnungsweite bei trockenem Wetter |
| Position bei schlechtem Wetter | 20% | Minimale Öffnung bei Regen/Sturm |

### 💧 Luftfeuchtigkeit
| Parameter | Standard | Beschreibung |
|---|---|---|
| Schwellwert | 75% | Ab diesem Wert öffnet die Luke |
| Hysterese | 10% | Luke schließt erst bei Schwellwert − Hysterese (= 65%) |

### 🌧️ Wetterschutz
| Parameter | Standard | Beschreibung |
|---|---|---|
| Wetterstation | – | Beliebige `weather`-Entität |
| Schlechte Zustände | Regen, Schnee, Gewitter, Sturm, Hagel | Minimalöffnung bei diesen Zuständen |

### ⏱️ Zeiten
| Parameter | Standard | Beschreibung |
|---|---|---|
| Maximale Öffnungsdauer | 45 min | Timer-basierte Zwangsschließung |
| Pause nach Schließen | 10 min | Automatisch aufgehoben |

### 🔔 Benachrichtigungen
| Kanal | Beschreibung |
|---|---|
| App-Push | notify-Dienst frei wählbar |
| Dashboard-Popup | `persistent_notification` + `input_text` für Statuskarte |
| Alexa | `notify.send_message` mit SSML; Lautstärke: x-soft / soft / medium / loud / x-loud |

---

## 🔄 Logik

```
Duschen beginnt
  → Feuchtigkeit > 75% + Licht an + kein Alarm + nicht deaktiviert
  → Luke öffnet (80% gut / 20% Regen) + Timer startet

Lüftung läuft...
  → Feuchtigkeit < 65% (75% − 10%)     → sofort schließen ✅
  → Timer läuft ab (45 min)             → Zwangsschließung ⏱️
  → Wetter ändert sich                  → Position korrigieren 🌧️
  → Alarm aktiviert                     → sofort schließen 🚨
  → Kurzer Druck Schließen              → Pause 10 min ⏸️
  → Langer Druck Schließen              → dauerhaft deaktiviert 🔴
  → Langer Druck Öffnen                 → Automation aktiv 🟢
  → Sensor nicht verfügbar              → keine auto. Öffnung, Timer läuft weiter
```

---

## 📊 Dashboard-Karte

```yaml
      - type: grid
        cards:
          - type: vertical-stack
            cards:
              - type: markdown
                content: >
                  {% set status = states('input_text.dachluke_status') %}
                  ### 🪟 Dachluke Status
                  {{ status if status else '✅ Keine aktiven Meldungen' }}
              - type: custom:mushroom-cover-card
                entity: cover.dachluke
                show_tilt_position_control: true
                show_position_control: true
                show_buttons_control: true
                icon_type: icon
                primary_info: state
                secondary_info: name
                fill_container: true
                layout: vertical
              - type: gauge
                entity: sensor.dachluke_ch3
                name: Öffnungsposition (Kanal 3)
                needle: false
                severity:
                  green: 0
                  yellow: 1
                  red: 90
                min: 0
                max: 100
              - type: gauge
                entity: sensor.og_bad_sensor_temperatur_luftdruck_humidity
                name: Luftfeuchtigkeit Bad
                needle: false
                severity:
                  green: 0
                  yellow: 65
                  red: 75
                min: 0
                max: 100
          - type: custom:bubble-card
            card_type: pop-up
            hash: '#dachluke-bad'
            show_header: false
            bg_opacity: '30'
```

---

## 📝 Changelog

### v4.0.0
- Komplett auf HA 2026.x Syntax aktualisiert (`triggers`/`actions` statt `trigger`/`action`)
- Nativer **Timer**-Helfer ersetzt `input_datetime` – schließt exakt bei 0, kein minütlicher Tick mehr
- Zweiter Blueprint (Timer-Wächter) entfällt – alles in einer Automation
- Wetterkorrektur während Lüftung ohne Timer-Reset
- Alexa-Lautstärke als Dropdown in Web-UI (x-soft / soft / medium / loud / x-loud)
- Optionale Sektionen (Alarm, Kontakt) standardmäßig zugeklappt

### v3.0.0
- Positionsüberwachung auf Kanal 3 umgestellt
- Fensterkontakt als optionale Plausibilitätsprüfung

### v2.0.0
- Event-Entitäten als Trigger statt Cover-State-Changes
- Langer Tastendruck zum Aktivieren/Deaktivieren
- SSML Lautstärke für Alexa

### v1.0.0
- Erstveröffentlichung

---

## 👤 Autor

**C4mp3r-Grey** – [github.com/C4mp3r-Grey](https://github.com/C4mp3r-Grey)

Feedback willkommen – Issues und Pull Requests sind offen.
