# Licht nach Zeit ohne Belegung oder Bewegung ausschalten

Home-Assistant-Blueprint (Domain `automation`), die Licht in einem Bereich automatisch
ausschaltet, wenn dort für eine einstellbare Wartezeit weder Belegung noch Bewegung
erkannt wurde. Prüft zusätzlich nach jedem Home-Assistant-Neustart.

## Installation

In Home Assistant: **Einstellungen → Automatisierungen → Blueprints → Blueprint importieren**
und die URL dieser Datei angeben:

```
https://github.com/rhammeran/licht-aus-nach-zeit-ohne-belegung-bewegung/blob/main/licht-aus-nach-zeit-ohne-belegung-bewegung.yaml
```

## Inputs

| Input | Pflicht | Beschreibung |
|---|---|---|
| `target_area` | ja | Bereich, in dem Licht, Belegung und Bewegung geprüft werden. |
| `timeout` | nein (Default 5 min) | Wartezeit ohne Belegung/Bewegung, nach der das Licht ausgeht. |
| `no_motion` | nein (Default aus) | Manueller Override: Bewegungsmelder bei der Prüfung ignorieren (z. B. defekter Sensor). Fehlt im Bereich ganz ein Bewegungsmelder, wird das automatisch erkannt – der Schalter ist dann nicht nötig. |
| `no_occupancy` | nein (Default aus) | Manueller Override: Belegungssensor bei der Prüfung ignorieren (z. B. defekter Sensor). Fehlt im Bereich ganz ein Belegungssensor, wird das automatisch erkannt – der Schalter ist dann nicht nötig. |

## Funktionsweise

- Löst aus, wenn ein Licht im Bereich angeht, wenn Belegung/Bewegung endet, oder nach
  einem Home-Assistant-Neustart.
- Schaltet das Licht nur aus, wenn es an ist **und** für die Wartezeit weder Belegung
  noch Bewegung im Bereich erkannt wurde.
- Erkennt automatisch per Template (`area_entities()` + `device_class`), ob im Bereich
  überhaupt ein Bewegungsmelder bzw. Belegungssensor existiert. Fehlt einer der beiden
  Sensortypen komplett, wird die entsprechende Bedingung automatisch als erfüllt
  gewertet – das Licht geht dann rein zeitbasiert nach der Wartezeit aus. Die
  `no_motion`/`no_occupancy`-Inputs musst du deshalb im Normalfall gar nicht setzen;
  sie dienen nur dazu, diese automatische Erkennung bei Bedarf manuell zu überschreiben
  (siehe Inputs-Tabelle oben).

## Lizenz

[MIT](LICENSE)

## Changelog

- **2026-08-15**: Automatische Erkennung fehlender Sensortypen per Template ergänzt
  (Root-Cause-Fix für stumm hängen bleibende Lichter in Räumen ohne Motion- oder
  Occupancy-Sensor). `no_motion`/`no_occupancy` sind seitdem optionale Overrides statt
  Pflicht-Schalter.
