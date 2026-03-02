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
- **Hysterese:** Start erst nach 3 Min. stabilem Überschuss, Stopp erst nach 5 Min. zu wenig
- **SOC-Schwellwert:** Überschussladen nur wenn PV-Batterie ausreichend geladen ist
- **Dashboard-Karte** mit Modus-Dropdown, SOC-Schieberegler und Auto-Status

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

#### Zahl (SOC-Schwellwert)
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

### Konfiguration je Blueprint

**Modus-Steuerung:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`

**Überschussladen – Starten:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- PV-Batterie SOC Sensor → FOX-ESS Batterie SOC (%)
- SOC-Schwellwert Helfer → `input_number.wallbox_soc_schwellwert`
- Anzahl Phasen → 3-phasig / 1-phasig
- Startschwelle → 4140 W *(3-phasig)* / 1380 W *(1-phasig)*
- Einschalt-Verzögerung → 00:03:00

**Überschussladen – Ampere anpassen:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Wallbox Ladestrom → `number.SERIAL_charge_amps`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- Aktuelle Ladeleistung → `sensor.SERIAL_current_energy`
- Anzahl Phasen → 3-phasig / 1-phasig

**Überschussladen – Stoppen:**
- Lademodus-Auswahl → `input_select.wallbox_modus`
- Wallbox Lade-Schalter → `switch.SERIAL_charge`
- Solar-Einspeisung Sensor → FOX-ESS Einspeisung (W)
- PV-Batterie SOC Sensor → FOX-ESS Batterie SOC (%)
- SOC-Schwellwert Helfer → `input_number.wallbox_soc_schwellwert`
- Stoppschwelle → 1380 W *(3-phasig)* / 460 W *(1-phasig)*
- Ausschalt-Verzögerung → 00:05:00

---

## Installation: Dashboard-Karte

1. Dashboard öffnen → **Bearbeiten** (Stift-Icon)
2. **+ Karte hinzufügen → Manuell (YAML-Modus)**
3. Inhalt von `dashboard/wallbox_card.yaml` einfügen
4. `SERIAL` durch deine Wallbox-Seriennummer ersetzen
5. Tesla- und BMW-Entity-IDs anpassen (oder im visuellen Editor per Dropdown wählen)
6. **Speichern**

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

## Lizenz

MIT
