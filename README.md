# Licht nach Zeit ohne Belegung oder Bewegung ausschalten

Home-Assistant-Blueprint (Domain `automation`), die Licht in einem Bereich automatisch
ausschaltet, wenn dort für eine einstellbare Wartezeit weder Belegung noch Bewegung
erkannt wurde. Optional werden Melder in Nachbarbereichen (z. B. Flur) mitgeprüft.
Prüft zusätzlich nach jedem Home-Assistant-Neustart.

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
| `neighbor_sensors` | nein (Default leer) | Bewegungs-/Belegungsmelder in Nachbarbereichen, z. B. im Flur. Meldet einer davon gerade Bewegung oder Belegung, bleibt das Licht an (es könnte jemand auf dem Weg in den Raum sein). |

## Funktionsweise

- Löst aus, wenn ein Licht im Bereich angeht, wenn Belegung/Bewegung endet, oder nach
  einem Home-Assistant-Neustart. Zusätzlich löst ein Template-Trigger aus, sobald im Bereich
  ein Licht an ist, auch wenn es nach einem Geräte-Neustart (z. B. Stromausfall, Wechsel
  `unavailable` → `on`) wieder angeht; der Trigger `light.turned_on` überspringt solche Wechsel.
- Schaltet das Licht nur aus, wenn es an ist **und** für die Wartezeit weder Belegung
  noch Bewegung im Bereich erkannt wurde.
- Erkennt automatisch per Template (`area_entities()` + `device_class`), ob im Bereich
  überhaupt ein Bewegungsmelder bzw. Belegungssensor existiert. Fehlt einer der beiden
  Sensortypen komplett, wird die entsprechende Bedingung automatisch als erfüllt
  gewertet – das Licht geht dann rein zeitbasiert nach der Wartezeit aus. Die
  `no_motion`/`no_occupancy`-Inputs musst du deshalb im Normalfall gar nicht setzen;
  sie dienen nur dazu, diese automatische Erkennung bei Bedarf manuell zu überschreiben
  (siehe Inputs-Tabelle oben).
- Nachbarmelder (`neighbor_sensors`): Solange einer davon Bewegung oder Belegung meldet,
  wird nicht ausgeschaltet. Meldet er frei, wird sofort nach den normalen Bedingungen
  geprüft – ohne erneute Wartezeit. Leer gelassen, verhält sich die Blueprint wie zuvor.
  Technischer Hinweis: Ein Trigger-Target darf in Home Assistant nicht leer sein, daher
  steht im Nachbar-Trigger zusätzlich der eigene Bereich; dessen Meldungen ohne Wartezeit
  scheitern an den Wartezeit-Bedingungen und schalten nichts.

## Lizenz

[MIT](LICENSE)

## Changelog

- **2026-08-15**: Automatische Erkennung fehlender Sensortypen per Template ergänzt
  (Root-Cause-Fix für stumm hängen bleibende Lichter in Räumen ohne Motion- oder
  Occupancy-Sensor). `no_motion`/`no_occupancy` sind seitdem optionale Overrides statt
  Pflicht-Schalter.
- **2026-08-15**: Input-Namen/-Beschreibungen präzisiert: „Kein Bewegungsmelder
  vorhanden" → „Kein Bewegungsmelder verwenden" (analog Belegung), da die Inputs
  jetzt einen bewussten Override beschreiben statt eines Hardware-Fakts.
- **2026-10-30**: Optionaler Input `neighbor_sensors` (Melder in Nachbarbereichen, z. B.
  Flur): Licht bleibt an, solange dort Bewegung/Belegung erkannt wird; bei „frei“ wird
  sofort geprüft. Ohne Eintrag unverändertes Verhalten.
- **2026-11-01**: Meldet ein Nachbarmelder „frei“, wird nur noch ausgeschaltet, wenn das
  Licht seit mindestens 2 Minuten an ist. Verhindert, dass frisch eingeschaltetes Licht sofort
  wieder ausgeht, bevor der Raummelder die Person erkannt hat.
- **2026-11-07**: Neuer Template-Trigger (`id: Wiederkehr`, mit `trigger_variables`): Geht ein Licht
  nach einem Geräte-Neustart (z. B. Stromausfall, Shelly mit „Schalter folgt Eingang“) von
  `unavailable` auf `on` und kommt danach kein Melder-Ereignis mehr, wird es nach der Wartezeit
  ausgeschaltet. Vorher blieb es an, weil `light.turned_on` Wechsel von `unavailable` überspringt.
