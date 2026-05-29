# 🪟 Dachluke Badentlüftung

Home Assistant Blueprint zur intelligenten Steuerung einer motorisierten Dachluke zur Badentlüftung – vollständig konfigurierbar über die Home Assistant Web-UI, kein YAML-Editieren erforderlich.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml)

---

## ✨ Features

- **Hygrometer-gesteuert** – öffnet bei hoher Luftfeuchtigkeit, schließt wenn Luft trocken (mit konfigurierbarer Hysterese)
- **Lichtsperre** – automatische Öffnung nur wenn Badezimmerlampe an ist; Trigger wenn Licht angeht damit hohe Feuchte nicht verpasst wird
- **Wetterabhängige Position** – weit bei gutem Wetter, minimal bei Regen/Sturm
- **Wetterkorrektur** – passt Position bei Wetterwechsel an, nur wenn Abweichung größer als konfigurierbare Toleranz; Timer wird nicht zurückgesetzt
- **Zuverlässige Positionsüberwachung** – über Kanal 3 des HmIP-BROLL statt unzuverlässigem Kanal 4
- **Zuverlässige Event-Trigger** – HmIP-BROLL Event-Entitäten statt Cover-State-Changes
- **Kein doppelter Cover-Befehl** – manuelle Bedienung wird erkannt, kein zusätzlicher Befehl gesendet
- **Kurzer Druck Öffnen** – manuelle Öffnung erkannt, Timer startet
- **Langer Druck Öffnen** – Automation aktivieren 🟢
- **Kurzer Druck Schließen** – manuelle Schließung erkannt, temporäre Pause startet
- **Langer Druck Schließen** – Automation dauerhaft deaktivieren 🔴
- **Pause ohne blockierendes delay** – Pause läuft über Trigger-basiertes Timeout ab
- **Getrennte Helfer** für temporäre Pause und dauerhafte Deaktivierung
- **Zuverlässige Zwangsschließung** – nativer HA-Timer, schließt exakt bei 0
- **Alarmo-Integration** – sperrt bei aktivem Alarm, schließt bei manuellem Öffnen während Alarm (optional)
- **Fensterkontakt** – optionale Plausibilitätsprüfung gegen Positionssensor
- **3 Benachrichtigungskanäle** – App-Push, Dashboard-Popup, Alexa-Sprachausgabe (SSML, Lautstärke und Umfang wählbar)

---

## 📋 Voraussetzungen

| Voraussetzung | Details |
|---|---|
| Home Assistant | mind. 2025.6.0 |
| Homematic(IP) Local | HACS-Integration, OpenCCU oder RaspberryMatic |
| Cover-Entität | Kanal 4 des HmIP-BROLL – nur für automatische Steuerbefehle |
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

Vier Helfer unter **Einstellungen → Helfer → Erstellen** anlegen:

| Typ | Name | Entity-ID | Zweck |
|---|---|---|---|
| Toggle | Dachluke Pause | `input_boolean.dachluke_pause` | Temporäre Pause nach kurzem Schließen |
| Toggle | Dachluke Deaktiviert | `input_boolean.dachluke_deaktiviert` | Dauerhafte Deaktivierung nach langem Schließen |
| Timer | Dachluke Lüftung | `timer.dachluke_lueftung` | Schließt exakt nach maximaler Öffnungsdauer |
| Text | Dachluke Status | `input_text.dachluke_status` | Dashboard-Statusanzeige (Max. 255 Zeichen) |

> ℹ️ Seit v0.5.0 sind Pause und Deaktivierung in **zwei getrennten Helfern** – das ermöglicht eine klare Unterscheidung zwischen temporärer Pause und dauerhafter Abschaltung.

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
| **Öffnen** | Manuelle Öffnung erkannt → Timer startet | Automation aktivieren 🟢 |
| **Schließen** | Manuelle Schließung erkannt → Pause startet ⏸️ | Automation dauerhaft deaktivieren 🔴 |

> ℹ️ Bei kurzen Drücken sendet die Automation **keinen zusätzlichen Cover-Befehl** – der Aktor hat bereits direkt reagiert.

---

## 🚀 Installation

### 1. Blueprint importieren
Auf den Badge oben klicken oder diese URL manuell unter **Einstellungen → Automatisierungen & Szenen → Blueprints → Blueprint importieren** eingeben:
```
https://raw.githubusercontent.com/C4mp3r-Grey/dachluke-badentlueftung/main/blueprints/automation/dachluke_steuerung.yaml
```

### 2. Kanal 3 aktivieren (siehe oben)

### 3. Vier Helfer anlegen (siehe oben)

### 4. Automation erstellen
**Einstellungen → Automatisierungen & Szenen → Blueprints → Dachluke Badentlüftung → Automatisierung erstellen**

---

## ⚙️ Konfiguration

### 🪟 Dachluke
| Parameter | Standard | Beschreibung |
|---|---|---|
| Cover-Entität | – | Steuerungsentität (Kanal 4), nur für automatische Befehle |
| Positionssensor | – | Kanal 3, z. B. `sensor.dachluke_ch3` |
| Position bei gutem Wetter | 80% | Öffnungsweite bei trockenem Wetter |
| Position bei schlechtem Wetter | 20% | Minimale Öffnung bei Regen/Sturm |
| Positions-Toleranz | 3% | Wetterkorrektur nur wenn Abweichung ≥ diesem Wert |

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
| Pause nach Schließen | 10 min | Automatisch aufgehoben via Trigger |

### 🔔 Benachrichtigungen
| Kanal | Beschreibung |
|---|---|
| App-Push | notify-Dienst frei wählbar |
| Dashboard-Popup | `persistent_notification` + `input_text` für Statuskarte |
| Alexa | `notify.send_message` mit SSML; Lautstärke: x-soft / soft / medium / loud / x-loud; Umfang: nur kritisch oder alle Änderungen |

---

## 🔄 Logik

```
Duschen beginnt
  → Feuchtigkeit > 75% + Licht an + kein Alarm + nicht deaktiviert
  → Luke öffnet (80% gut / 20% Regen) + Timer startet
  → Kein Cover-Befehl wenn Luke schon offen, nur Timer

Lüftung läuft...
  → Feuchtigkeit < 65% (75% − 10%)     → sofort schließen ✅
  → Timer läuft ab (45 min)             → Zwangsschließung ⏱️
  → Wetter ändert sich                  → Position korrigieren wenn Δ ≥ 3% 🌧️
  → Alarm aktiviert                     → sofort schließen 🚨
  → Manuell geöffnet während Alarm      → sofort wieder schließen 🚨
  → Kurzer Druck Schließen              → Pause 10 min erkannt ⏸️
  → Langer Druck Schließen              → dauerhaft deaktiviert 🔴
  → Langer Druck Öffnen                 → Automation aktiv 🟢
  → Sensor nicht verfügbar              → keine feuchtebasierte Aktion
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

### v0.5.1
- `sensor_available` verwendet `is_number()` statt Sentinel-Wert
- Remote-Attribut-Auswertung mit `| default('', true)` abgesichert
- `max: 3` statt `max: 10` bei `queued` mode

### v0.5.0
- **Zwei getrennte Helfer** für temporäre Pause (`input_boolean.dachluke_pause`) und dauerhafte Deaktivierung (`input_boolean.dachluke_deaktiviert`)
- **Kein doppelter Cover-Befehl** bei manueller Bedienung – Aktor hat bereits direkt reagiert
- **Pause-Ablauf via Trigger** statt blockierendem `delay` – Automation bleibt reaktionsfähig
- **`position_tolerance`** verhindert unnötige Motorbefehle bei Wetterkorrektur
- **`target_differs`** Variable für saubere Toleranz-Prüfung
- **`alexa_all` vs `alexa_critical`** – konfigurierbarer Alexa-Ansageumfang
- **Remote per `state_changed`-Event** – löst Problem mit leeren Entity-IDs elegant
- **Licht-Trigger** damit bereits hohe Feuchte nicht verpasst wird wenn Licht angeht
- **Manuelles Öffnen während Alarm** → sofort wieder schließen

### v0.4.1
- `alexa_ready` Variable verhindert Entity-ID-Fehler bei leerem Alexa-Feld

### v0.4.0
- Komplette Syntax-Überarbeitung für HA 2026.x
- Nativer Timer-Helfer ersetzt `input_datetime`
- Ein Blueprint statt zwei
- Wetterkorrektur ohne Timer-Reset
- Alexa-Lautstärke als Dropdown

### v0.3.0
- Positionsüberwachung auf Kanal 3 umgestellt
- Fensterkontakt als optionale Plausibilitätsprüfung

### v0.2.0
- Event-Entitäten als Trigger statt Cover-State-Changes
- Langer Tastendruck zum Aktivieren/Deaktivieren
- SSML Lautstärke für Alexa

### v0.1.0
- Erstveröffentlichung

---

## 👤 Autor

**C4mp3r-Grey** – [github.com/C4mp3r-Grey](https://github.com/C4mp3r-Grey)

Feedback willkommen – Issues und Pull Requests sind offen.
