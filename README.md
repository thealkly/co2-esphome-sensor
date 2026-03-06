# CO2 Traffic Light with SCD41 & NeoPixel – ESPHome

A smart CO2 traffic light built with an **ESP32**, the **Sensirion SCD41** CO2/temperature/humidity sensor, and **WS2812B NeoPixel** LEDs. Fully local via **Home Assistant** – no cloud dependency.




## Deutsche Version UNTEN

## Features

- **Real-time CO2 measurement** with the Sensirion SCD41 (400–5000 ppm)
- **Color-coded LED indicator** via WS2812B NeoPixel strip
- **Temperature & humidity** as bonus sensors
- **Text-based air quality rating** directly in Home Assistant
- **LED on/off switch** – e.g. turn off at night via automation
- **Manual calibration** (FRC) button from Home Assistant
- **Altitude compensation** pre-configured
- **Boot animation** – pulsing blue until first readings arrive

## CO2 Levels & Colors

| CO2 (ppm) | Color | Brightness | Rating |
|-----------|-------|-----------|--------|
| < 800 | 🟢 Green | 30% | Excellent |
| 800 – 1000 | 🟡 Yellow | 50% | Good – ventilate soon |
| 1000 – 1400 | 🟠 Orange | 70% | Moderate – open windows! |
| 1400 – 2000 | 🔴 Red | 100% | Poor – ventilate urgently! |
| > 2000 | 🔴 Red flashing | 100% | Critical – ventilate now! |

## Hardware

### Bill of Materials

CO2 Sensor https://amzn.to/4ucHS8e *
ESP32 C3 Super Mini https://amzn.to/4sqjfmU *
Display https://amzn.to/40N0oGQ * 
Neopix https://amzn.to/4rOGJCj *
Schrauben Set https://amzn.to/4smAsO3 *

Links marked with ‘*’ are promotional links -> Affiliate links to Amazon. If you use them, I earn a commission at no extra cost to you. Thanks!

### Wiring

```
ESP32 DevKit v1
┌─────────────────────┐
│                     │
│  GPIO21 (SDA) ──────┼──── SCD41 SDA
│  GPIO22 (SCL) ──────┼──── SCD41 SCL
│  3.3V ──────────────┼──── SCD41 VCC
│                     │
│  GPIO16 ────────────┼──── NeoPixel DIN
│  5V (VIN) ──────────┼──── NeoPixel VCC
│                     │
│  GND ───────────────┼──── SCD41 GND + NeoPixel GND
│                     │
└─────────────────────┘
```
![3 6 CO2 Sensor v 01_Steckplatine](https://github.com/user-attachments/assets/7adef368-ba31-47b0-ac99-313a7f073688)


## Installation

### Prerequisites

- [ESPHome](https://esphome.io/) installed (as Home Assistant add-on or CLI)
- Home Assistant with ESPHome integration
- `secrets.yaml` with your WiFi credentials

### 1. Clone the repository

```bash
git clone https://github.com/DEIN_USERNAME/co2-ampel-esphome.git
cd co2-ampel-esphome
```

### 2. Create secrets file

Create a `secrets.yaml` in your ESPHome directory (if not already present):

```yaml
wifi_ssid: "YOUR_WIFI_NAME"
wifi_password: "YOUR_WIFI_PASSWORD"
```

### 3. Customize configuration

Open `co2_ampel_scd41.yaml` and adjust if needed:

```yaml
substitutions:
  neopixel_pin: GPIO16    # Your NeoPixel data pin
  num_leds: "8"           # Number of LEDs
  sda_pin: GPIO21         # I²C SDA
  scl_pin: GPIO22         # I²C SCL
```

**Altitude compensation** (in the `sensor:` block):
```yaml
altitude_compensation: 400m   # Your altitude above sea level
```

**Temperature offset** (SCD41 self-heating):
```yaml
filters:
  - offset: -3.0   # Adjust after comparing with a reference thermometer
```

### 4. Flash

```bash
esphome run co2_ampel_scd41.yaml
```

Or use the ESPHome Dashboard in Home Assistant.

## Home Assistant Integration

After flashing, the device appears automatically in Home Assistant. The following entities are created:

| Entity | Type | Description |
|--------|------|-------------|
| `sensor.co2_ampel_co2` | Sensor | CO2 in ppm |
| `sensor.co2_ampel_temperatur` | Sensor | Temperature in °C |
| `sensor.co2_ampel_luftfeuchtigkeit` | Sensor | Relative humidity in % |
| `text_sensor.co2_ampel_luftqualitaet` | Text | Air quality rating as text |
| `light.co2_ampel_led` | Light | NeoPixel LED (manual control) |
| `switch.co2_ampel_led_aktiv` | Switch | LED automation on/off |
| `button.co2_ampel_neustart` | Button | ESP32 restart |
| `button.co2_ampel_co2_kalibrierung_frc` | Button | Manual calibration |


## Calibration

### Automatic Self-Calibration (ASC)

Enabled by default. The SCD41 calibrates itself automatically when regularly exposed to fresh air (e.g. ventilated room). Takes approximately 7 days for initial calibration.

### Forced Recalibration (FRC)

1. Let the sensor run for at least **3 minutes** with fresh air (window wide open)
2. Press the **"CO2 Ampel CO2 Kalibrierung (FRC)"** button in Home Assistant
3. The sensor calibrates to 420 ppm (current outdoor CO2 reference value)

> ⚠️ **Warning:** Incorrect calibration leads to inaccurate readings! When in doubt, use ASC.

## Customization

### Different LED types

For SK6812 (RGBW), change the `type`:
```yaml
light:
  - platform: neopixelbus
    type: GRBW
    variant: SK6812
```

### More LEDs / LED ring

Simply adjust `num_leds` in substitutions:
```yaml
substitutions:
  num_leds: "16"   # e.g. for a 16-LED NeoPixel ring
```

### Integration into an existing project

The relevant blocks (`i2c`, `sensor: scd4x`, `light: neopixelbus`, `script`, `interval`) can be copied into any existing ESPHome YAML. Watch out for pin conflicts!

## Troubleshooting

| Problem | Solution |
|---------|----------|
| SCD41 not found | Check I²C scan in logs, verify wiring (address: 0x62) |
| CO2 values unrealistically high/low | Perform manual calibration with fresh air |
| LEDs show wrong colors | Change `type: GRB` to `RGB` (depends on LED type) |
| LEDs flicker | Add 330Ω resistor on data pin, capacitor on VCC/GND |
| Temperature too high | Adjust `offset` in temperature filter |
| ESP32 won't boot | GPIO16 is safe; avoid GPIO12 (strapping pin) |


## License

This project is licensed under the [MIT License](LICENSE).

---

# 🚦 CO2-Ampel mit SCD41 & NeoPixel – ESPHome

Eine smarte CO2-Ampel auf Basis eines **ESP32**, dem **Sensirion SCD41** CO2-/Temperatur-/Feuchte-Sensor und **WS2812B NeoPixel** LEDs. Vollständig lokal über **Home Assistant** steuerbar – kein Cloud-Zwang.


> 💡 **Du willst tiefer einsteigen?** Schau dir den [ESPHome Meisterkurs](https://alkly.de/esphome-meisterkurs/?utm_source=github&utm_medium=github&utm_campaign=meisterkurs&utm_content=CO2Sensor) an – der umfassende deutschsprachige Kurs rund um lokale Smart-Home-Geräte mit ESP32 und Home Assistant.

---

## Funktionen

- **Echtzeit CO2-Messung** mit dem Sensirion SCD41 (400–5000 ppm)
- **Farbcodierte LED-Anzeige** über WS2812B NeoPixel-Streifen
- **Temperatur & Luftfeuchtigkeit** als Bonus-Sensoren
- **Text-Bewertung** der Luftqualität direkt in Home Assistant
- **LED ein/aus Schalter** – z.B. nachts per Automation ausschalten
- **Manuelle Kalibrierung** (FRC) per Button aus Home Assistant
- **Höhenkompensation** vorkonfiguriert
- **Boot-Animation** – Blau pulsierend bis erste Messwerte vorliegen

## CO2-Ampel Stufen

| CO2 (ppm) | Farbe | Helligkeit | Bewertung |
|-----------|-------|-----------|-----------|
| < 800 | 🟢 Grün | 30% | Sehr gut |
| 800 – 1000 | 🟡 Gelb | 50% | Gut – bald lüften |
| 1000 – 1400 | 🟠 Orange | 70% | Mäßig – Lüften! |
| 1400 – 2000 | 🔴 Rot | 100% | Schlecht – Dringend lüften! |
| > 2000 | 🔴 Rot blinkend | 100% | Kritisch – Sofort lüften! |

## Hardware

### Bauteile

| Bauteil | Beschreibung | ca. Preis |
|---------|-------------|-----------|

CO2 Sensor https://amzn.to/4ucHS8e *
ESP32 C3 Super Mini https://amzn.to/4sqjfmU *
Display https://amzn.to/40N0oGQ * 
Neopix https://amzn.to/4rOGJCj *
Schrauben Set https://amzn.to/4smAsO3 *
Links mit „*“ sind Werbe Links -> Affiliate-Links zu Amazon. Bei Nutzung erhalte ich eine Provision, ohne dass dir Mehrkosten entstehen. Danke!

Gehäuse gibt es in der [Macherwerkstatt](https://alkly.de/die-macherwerkstatt/?utm_source=github&utm_medium=github&utm_campaign=macherwerkstatt-live&utm_content=CO2Sensor) (monatliche Live Calls und 3D Druck Bibliothek) oder in dem [ESPHome Meisterkurs](https://alkly.de/esphome-meisterkurs/?utm_source=github&utm_medium=github&utm_campaign=meisterkurs&utm_content=CO2Sensor)

### Verkabelung

![3 6 CO2 Sensor v 01_Steckplatine](https://github.com/user-attachments/assets/8e56fdd2-8688-421e-b22b-939e065154b8)



> **Hinweis:** Bei mehr als 8 LEDs unbedingt separate 5V-Versorgung nutzen und einen 330Ω Widerstand am Datenpin sowie einen 1000µF Elko an der LED-Versorgung vorsehen.

## Installation

### Voraussetzungen

- [ESPHome](https://esphome.io/) installiert (als Home Assistant Add-on oder CLI)
- Home Assistant mit ESPHome-Integration
- `secrets.yaml` mit deinen WLAN-Zugangsdaten

### 1. Repository klonen

```bash
git clone https://github.com/DEIN_USERNAME/co2-ampel-esphome.git
cd co2-ampel-esphome
```

### 2. Secrets anlegen

Erstelle eine `secrets.yaml` im ESPHome-Verzeichnis (falls nicht vorhanden):

```yaml
wifi_ssid: "DEIN_WLAN_NAME"
wifi_password: "DEIN_WLAN_PASSWORT"
api_key: "DEIN_API_KEY"          # esphome generate-api-key
ota_password: "DEIN_OTA_PASSWORT"
ap_password: "DEIN_AP_PASSWORT"
```

### 3. Konfiguration anpassen

Öffne `co2_ampel_scd41.yaml` und passe ggf. an:

```yaml
substitutions:
  neopixel_pin: GPIO16    # Dein NeoPixel Data-Pin
  num_leds: "8"           # Anzahl deiner LEDs
  sda_pin: GPIO21         # I²C SDA
  scl_pin: GPIO22         # I²C SCL
```

**Höhenkompensation** (im `sensor:` Block):
```yaml
altitude_compensation: 400m   # Deine Höhe über NN
```

**Temperatur-Offset** (SCD41 Eigenerwärmung):
```yaml
filters:
  - offset: -3.0   # Ggf. anpassen nach Vergleichsmessung
```

### 4. Flashen

```bash
esphome run co2_ampel_scd41.yaml
```

Oder über das ESPHome Dashboard in Home Assistant.

## Home Assistant Integration

Nach dem Flashen erscheint das Gerät automatisch in Home Assistant. Folgende Entities werden erstellt:

| Entity | Typ | Beschreibung |
|--------|-----|-------------|
| `sensor.co2_ampel_co2` | Sensor | CO2 in ppm |
| `sensor.co2_ampel_temperatur` | Sensor | Temperatur in °C |
| `sensor.co2_ampel_luftfeuchtigkeit` | Sensor | Relative Luftfeuchtigkeit in % |
| `text_sensor.co2_ampel_luftqualitaet` | Text | Bewertung als Text |
| `light.co2_ampel_led` | Light | NeoPixel LED (manuell steuerbar) |
| `switch.co2_ampel_led_aktiv` | Switch | LED-Automatik ein/aus |
| `button.co2_ampel_neustart` | Button | ESP32 Neustart |
| `button.co2_ampel_co2_kalibrierung_frc` | Button | Manuelle Kalibrierung |


## Kalibrierung

### Automatische Selbstkalibrierung (ASC)

Ist standardmäßig **aktiviert**. Der SCD41 kalibriert sich automatisch, wenn er regelmäßig frischer Außenluft ausgesetzt wird (z.B. bei gelüftetem Raum). Braucht ca. 7 Tage für die erste Kalibrierung.

### Manuelle Kalibrierung (FRC)

1. Sensor mindestens **3 Minuten** bei Frischluft (Fenster weit auf) laufen lassen
2. In Home Assistant den Button **"CO2 Ampel CO2 Kalibrierung (FRC)"** drücken
3. Der Sensor wird auf 420 ppm kalibriert (aktueller Außenluft-Referenzwert)

> ⚠️ **Achtung:** Falsche Kalibrierung führt zu ungenauen Werten! Im Zweifel ASC nutzen.

## Anpassungen

### Andere LED-Typen

Für SK6812 (RGBW) den `type` ändern:
```yaml
light:
  - platform: neopixelbus
    type: GRBW
    variant: SK6812
```

### Mehr LEDs / LED-Ring

Einfach `num_leds` in den Substitutions anpassen:
```yaml
substitutions:
  num_leds: "16"   # z.B. für einen 16er NeoPixel Ring
```

### Integration in bestehendes Projekt

Die relevanten Blöcke (`i2c`, `sensor: scd4x`, `light: neopixelbus`, `script`, `interval`) können in jede bestehende ESPHome-YAML übernommen werden. Auf Pin-Konflikte achten!

## Fehlerbehebung

| Problem | Lösung |
|---------|--------|
| SCD41 wird nicht gefunden | I²C-Scan im Log prüfen, Verkabelung checken (Adresse: 0x62) |
| CO2-Werte unrealistisch hoch/niedrig | Manuelle Kalibrierung bei Frischluft durchführen |
| LEDs zeigen falsche Farben | `type: GRB` ggf. auf `RGB` ändern (je nach LED-Typ) |
| LEDs flackern | 330Ω Widerstand am Datenpin, Kondensator an VCC/GND |
| Temperatur zu hoch | `offset` im Temperatur-Filter anpassen |
| ESP32 bootet nicht | GPIO16 ist safe, aber GPIO12 (Strapping Pin) vermeiden |



## Mitmachen

Pull Requests und Issues sind willkommen!

1. Fork erstellen
2. Feature-Branch anlegen (`git checkout -b feature/mein-feature`)
3. Änderungen committen (`git commit -m 'feat: Beschreibung'`)
4. Branch pushen (`git push origin feature/mein-feature`)
5. Pull Request erstellen

## Lizenz

Dieses Projekt steht unter der [MIT-Lizenz](LICENSE).

---

> 💡 **Mehr lernen:** Dieses Projekt ist Teil des [ESPHome Meisterkurs](https://esphome-meisterkurs.de) Ökosystems – der praxisnahe Kurs, mit dem du lokale Smart-Home-Geräte mit ESP32 und Home Assistant baust.
