# pv_automatic_charging

Home Assistant Blueprints für automatisches Laden einer Wallbox mit Solar-Überschuss.

Entwickelt für:
- **Wallbox:** Über das [evseMQTT](https://github.com/david120378/evsemqtt-ha) HA Add-on eingebunden
- **PV-Anlage:** FOX-ESS (oder andere Anlagen mit Einspeisung- und SOC-Sensor)
- **Autos:** Tesla, BMW/Mini ConnectedDrive (oder andere HA-Integrationen)

---

## Features

- **3 Lademodi:** Off / Manuell / Überschussladen
- **Automatische Ampere-Regelung** im Überschussladen-Modus
- **Einspeisung-Puffer:** Stabilisiert die Regelung bei schwankender Solarproduktion
- **Lastmanagement:** Optionaler Netzbezug- und Großverbraucher-Sensor für präzise Regelung
- **Hysterese:** Start erst nach 3 Min. stabilem Überschuss, Stopp erst nach 5 Min. zu wenig
- **SOC-Schwellwert:** Überschussladen nur wenn PV-Batterie ausreichend geladen ist
- **Mindest-SOC Auto:** Laden stoppt wenn Auto den Ziel-SOC erreicht (Akku-Schonung)
- **Zeitfenster:** Überschussladen optional nur in definierten Zeiten (z.B. 08:00–20:00)
- **Notfall-Laden:** Automatisch laden wenn Auto-SOC unter Schwellwert fällt
- **Wetter-Vorschau:** Bei schlechtem Wetter → Nachtladung automatisch aktivieren
- **Benachrichtigungen:** Optional für alle Modusänderungen und Ereignisse

---

## Voraussetzungen

### 1. Helfer in HA erstellen

Gehe zu **Einstellungen → Geräte & Dienste → Helfer → + Helfer erstellen**:

#### Dropdown (Lademodus)
| Feld | Wert |
|---|---|
| Typ | Dropdown |
| Name | `Wallbox Modus` |
| Entity-ID | `input_select.wallbox_modus` *(automatisch)* |
| Optionen | `Off` · `Manuell` · `Überschussladen` |

#### Zahl (SOC-Schwellwert PV-Batterie)
| Feld | Wert |
|---|---|
| Typ | Zahl |
| Name | `Wallbox SOC Schwellwert` |
| Entity-ID | `input_number.wallbox_soc_schwellwert` *(automatisch)* |
| Minimum | `0` |
| Maximum | `100` |
| Schrittweite | `5` |
| Einheit | `%` |
| Icon | `mdi:battery-charging` |

#### Zahl (Einspeisung-Puffer) — optional, für surplus_start + surplus_amps
| Feld | Wert |
|---|---|
| Typ | Zahl |
| Name | `Wallbox Puffer Watt` |
| Entity-ID | `input_number.wallbox_puffer_watt` *(automatisch)* |
| Minimum | `0` |
| Maximum | `1000` |
| Schrittweite | `50` |
| Einheit | `W` |
| Icon | `mdi:sine-wave` |

#### Schalter (Nacht-Laden-Flag) — nur für wallbox_wetter Blueprint
| Feld | Wert |
|---|---|
| Typ | Schalter |
| Name | `Wallbox Nacht Laden` |
| Entity-ID | `input_boolean.wallbox_nacht_laden` *(automatisch)* |

#### Zahl (Nacht-Ziel-SOC) — nur für wallbox_wetter Blueprint
| Feld | Wert |
|---|---|
| Typ | Zahl |
| Name | `Wallbox Nacht Ziel SOC` |
| Entity-ID | `input_number.wallbox_nacht_ziel_soc` *(automatisch)* |
| Minimum | `0` |
| Maximum | `100` |
| Schrittweite | `5` |
| Einheit | `%` |

---

## Installation: Blueprints

Für jeden Blueprint:
1. **Einstellungen → Automationen & Szenen → Blueprints**
2. **Blueprint importieren** (blauer Button unten rechts)
3. GitHub Raw-URL einfügen (siehe unten)
4. **Vorschau → Importieren**
5. Anschließend: **Automation aus Blueprint erstellen** und Entitäten per Dropdown auswählen

### Blueprint-URLs

Ersetze `david120378/pv_automatic_charging` durch dein Repository wenn nötig.

| Blueprint | URL |
|---|---|
| Modus (Off/Manuell) | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_modus.yaml` |
| Überschussladen – Starten | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_surplus_start.yaml` |
| Überschussladen – Ampere | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_surplus_amps.yaml` |
| Überschussladen – Stoppen | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_surplus_stop.yaml` |
| Notfall-Laden | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_notfall.yaml` |
| Wetter-Vorschau & Nachtladung | `https://raw.githubusercontent.com/david120378/pv_automatic_charging/main/blueprints/wallbox_wetter.yaml` |

### Konfiguration je Blueprint

**Modus-Steuerung:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Ladestrom im Manuell-Modus → 16 A *(Standard)*
- Benachrichtigungs-Dienst → optional (z.B. `notify.mobile_app_iphone`)

**Überschussladen – Starten:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- PV-Batterie SOC Sensor → FOX-ESS Batterie SOC (%)
- SOC-Schwellwert Helfer → `input_number.wallbox_soc_schwellwert`
- Anzahl Phasen → 3-phasig / 1-phasig
- Startschwelle → 4140 W *(3-phasig)* / 1380 W *(1-phasig)*
- Einspeisung-Puffer → 200 W *(Standard)*
- Einschalt-Verzögerung → 00:03:00
- Zeitfenster → optional aktivierbar (z.B. 08:00–20:00)
- Benachrichtigungs-Dienst → optional

**Überschussladen – Ampere anpassen:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- Aktuelle Ladeleistung → `sensor.SERIAL_current_energy`
- Anzahl Phasen → 3-phasig / 1-phasig
- Einspeisung-Puffer → 200 W *(Standard)*
- Netzbezug-Sensor → optional (z.B. `sensor.fox_ess_grid_consumption_power`)
- Großverbraucher-Sensor → optional (Template-Sensor der alle Großverbraucher summiert)
- Benachrichtigungs-Dienst → optional (nur bei Änderung ≥ 2A)

**Überschussladen – Stoppen:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- PV-Batterie SOC Sensor → FOX-ESS Batterie SOC (%)
- SOC-Schwellwert Helfer → `input_number.wallbox_soc_schwellwert`
- Stoppschwelle → 1380 W *(3-phasig)* / 460 W *(1-phasig)*
- Ausschalt-Verzögerung (Einspeisung) → 00:05:00
- Ausschalt-Verzögerung (SOC) → 00:01:00
- Auto SOC Sensor → optional (z.B. `sensor.tesla_battery_level`)
- Auto Ziel-SOC → 80 % *(Standard, zum Akku-Schutz)*
- Benachrichtigungs-Dienst → optional

**Notfall-Laden:**
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Verbindungs-Status Sensor → `sensor.SERIAL_plug_state`
- Auto SOC Sensor → z.B. `sensor.tesla_battery_level`
- Notfall-Schwellwert → 20 % *(Standard)*
- Erholungs-Schwellwert → 30 % *(Standard)*
- Notfall-Ladestrom → 6 A *(Standard)*
- Benachrichtigungs-Dienst → optional
- *(Für jedes Auto eine eigene Automation erstellen)*

**Wetter-Vorschau & Nachtladung:**
- Wetter-Entity → z.B. `weather.home`
- Prüfzeit → 17:00 *(täglich)*
- Nachtladung Start → 01:00
- Nachtladung Ende → 05:00
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Nacht-Ladestrom → 16 A *(Standard)*
- Nachtladung-Flag Helfer → `input_boolean.wallbox_nacht_laden`
- Nacht-Ziel-SOC Helfer → `input_number.wallbox_nacht_ziel_soc`
- Auto SOC Sensor → optional
- Benachrichtigungs-Dienst → optional
- ⚠️ **Benötigt Home Assistant 2023.9+** (für `weather.get_forecasts`)

---

## Hysterese-Erklärung

```
Einspeisung (W)
     │
4140 ┤─────────── Startschwelle (Laden beginnt nach 3 Min.)
     │
1380 ┤─────────── Stoppschwelle (Laden stoppt nach 5 Min.)
     │
   0 ┤
```

Das Band zwischen Start- und Stoppschwelle verhindert ständiges Ein-/Ausschalten
bei schwankender Solarproduktion (z.B. bei wechselhafter Bewölkung).

---

## Optionale Sensoren (Lastmanagement)

Der `wallbox_surplus_amps` Blueprint unterstützt zwei optionale Sensoren für präzisere Regelung:

- **Netzbezug-Sensor** (`grid_sensor`): Berücksichtigt aktuelle Netzentnahme anderer Verbraucher.
  Der Wert wird von der verfügbaren Leistung abgezogen, sodass nicht versehentlich Strom
  aus dem Netz für das Laden verwendet wird.
  → Empfohlen: FOX-ESS Grid Consumption Power Sensor

- **Großverbraucher-Sensor** (`extra_load_sensor`): Template-Sensor der die Summe der
  Leistung aller Großverbraucher (z.B. Wärmepumpe + Geschirrspüler) anzeigt.
  → Beispiel: `{{ states('sensor.waermepumpe_power') | float(0) + states('sensor.geschirrspueler_power') | float(0) }}`

**Formel:**
```
Verfügbar = Einspeisung + Ladeleistung − Netzbezug − Großverbraucher − Puffer
```

---

## Hinweise

- **Mindest-Ladestrom:** 6A (EVSE-Standard). Ampere werden nie unter 6A gesetzt.
- **Für jedes Auto separat:** Die Blueprints `wallbox_surplus_stop` und `wallbox_notfall`
  unterstützen einen Auto-SOC-Sensor. Für mehrere Autos je eine eigene Automation erstellen.
- **Tibber-Integration:** Für preisbasiertes Laden (Niedertarif-Stunden) kann der
  `wallbox_wetter` Blueprint als Vorlage für eine eigene Tibber-gesteuerte Nachtladung dienen.

---

## Lizenz

MIT
