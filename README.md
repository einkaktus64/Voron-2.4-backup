# 🚀 Voron 2.4 – Klipper Config Doku

> Dokumentation der Live-Konfiguration (Repo: `Voron-2.4-backup`, Backup via [klipper-backup](https://github.com/Staubgeborener/klipper-backup))

---

## 🖨️ Hardware-Überblick

| Komponente | Details |
|---|---|
| Kinematik | CoreXY, 4x unabhängige Z-Motoren (QGL) |
| Mainboard (MCU) | STM32G0B1, USB-Anbindung (`stm32g0b1xx`) |
| Toolhead-Board (MCU `nhk`) | RP2040-basiert (nhk = wahrscheinlich BTT/EBB-Klon), USB angebunden |
| Extruder | Galileo 2 (Direct Drive, Gear-Ratio 9:1) |
| Hotend/Probe | Chaos Labs CNC Tap (Klick-Endstop über Nozzle) |
| Bett-Heizung | SSR-geschaltet, max_power 0.6 |
| Steuerung/UI | Mainsail, KlipperScreen |
| Kammer-Sensor | Generic 3950 Thermistor |
| Beleuchtung | 2x Neopixel-Ketten (Chamber 28 LEDs, Display 7 LEDs) |
| Bauraum | 250×250×220 mm effektiv (X/Y 0–250, Z bis 220) |

Bemerkenswert: `[mcu EBBCan]` (CAN-Toolhead-Board) ist im Config auskommentiert – aktuell läuft der Toolhead offenbar über die USB-verbundene `nhk`-Platine statt CAN-Bus.

---

## ⚙️ Steppers & Antrieb

| Achse | Treiber | Microsteps | Rotation Distance | Homing |
|---|---|---|---|---|
| X | TMC2209, 0.8 A | 32 | 40 mm | **Sensorless** (StallGuard, SGTHRS 140) |
| Y | TMC2209, 0.8 A | 32 | 40 mm | **Sensorless** (StallGuard, SGTHRS 141) |
| Z0–Z3 | TMC2209, 0.8 A | 64 | 40 mm, Gear-Ratio 80:16 | Probe (Tap) |
| Extruder | TMC2209, 0.6 A | 16 | 47.088 mm (Galileo 2), Gear-Ratio 9:1 | – |

- **Sensorless Homing** für X/Y ist aktiv – dafür gibt's ein eigenes `homing.cfg` mit `_HOME_X`/`_HOME_Y`-Makros, die vor dem Homing kurzzeitig den Stepperstrom auf 0.49 A absenken (schont die Motoren/Treiber beim StallGuard-Trigger) und danach wieder auf den Run-Current zurückstellen.
- `homing_override` sorgt dafür, dass beim vollen `G28` zuerst Z virtuell "gehomed" wird (`SET_KINEMATIC_POSITION`), dann X/Y einzeln über die Custom-Makros, und erst danach die Z-Achse via Tap-Probe in der Bettmitte.
- `force_move: enable_force_move: True` ist aktiv – nötig für die `SET_KINEMATIC_POSITION`-Tricks beim Sensorless Homing.

---

## 🎯 Probe (Chaos Labs Tap)

```
[probe]
pin: ~!nhk:gpio10
x_offset: 0
y_offset: 0
speed: 3.5
samples: 3
samples_result: median
samples_tolerance: 0.02
```

- Aktueller **Z-Offset: -0.640 mm** (im SAVE_CONFIG-Block gespeichert)
- Vor dem Probing wird die Hotend-Temperatur automatisch auf **150 °C** begrenzt (`activate_gcode`), um thermische Ausdehnung beim Homing zu vermeiden
- Bed Mesh: 6×6 Punkte, Bereich 25–225 mm, Lagrange-Interpolation

---

## 📐 Quad Gantry Level

```
gantry_corners: -60,-10 / 310,320
points: 25,25 / 25,225 / 225,225 / 225,25
retries: 5, retry_tolerance: 0.015, max_adjust: 10
```

Zusätzlich gibt's ein `_FINE_QUAD_GANTRY_LEVEL`-Makro in `macros.cfg` für einen zweiten, feineren QGL-Durchgang.

---

## 🌀 Lüfter & Temperaturregelung

| Lüfter | Pin | Zweck |
|---|---|---|
| `fan` (Part Cooling) | `nhk:gpio6` | Bauteilkühlung |
| `hotend_fan` | `nhk:gpio5` | Hotend-Kühlung, ab 50 °C |
| `MCUTemp+_fan` | PE0 | PID-geregelt, Ziel 45 °C (Mainboard-Kühlung) |
| `CM4` (temperature_fan) | PB9 | PID-geregelt, Ziel 45 °C (SBC/CM4-Kühlung) |
| `nevermore` | PE6 | Manuell geschaltet (`output_pin`), kein eigener PWM-Fan mehr |

Controller-Fan und Exhaust-Fan sind im Config auskommentiert (nicht verbaut/nicht genutzt).

---

## 💡 PID-Werte (aus SAVE_CONFIG)

| Heizer | Kp | Ki | Kd |
|---|---|---|---|
| Heater Bed | 37.493 | 1.344 | 261.514 |
| Extruder | 27.992 | 1.236 | 158.506 |

---

## 📳 Input Shaping & Skew

- `shaper_type_x/y: mzv`, X-Freq **62.2 Hz**, Y-Freq **43.4 Hz**
- ADXL345 am Toolhead (SPI Software-Pins über `nhk`)
- Skew-Korrektur aktiv, Profil `CaliFlower` (xy_skew ≈ 0.0003, praktisch vernachlässigbar)
- **Klippain Shake&Tune** ist eingebunden für Resonanz-Analyse-Reports

---

## 📜 Makro-Übersicht (`macros.cfg`)

| Makro | Zweck |
|---|---|
| `PARK` / `PAUSE` / `RESUME` / `_TOOLHEAD_PARK_PAUSE_CANCEL` | Pause/Resume-Handling mit Park-Position |
| `PRINT_START` | Homing, bedingter Heatsoak (>90 °C Bett), Nozzle-Clean, QGL, Mesh/Skew laden, Aufheizen |
| `END_PRINT` | Druckende, Heizer aus, Wipe, Steppers aus |
| `CANCEL_PRINT` | Druckabbruch-Routine |
| `LOAD_FILAMENT` / `UNLOAD_FILAMENT` | Filamentwechsel |
| `QUAD_GANTRY_LEVEL` / `_FINE_QUAD_GANTRY_LEVEL` | QGL-Wrapper |
| `PID_EXTRUDER` / `PID_BED` | PID-Tuning-Shortcuts |
| `clean_nozzle` | Nozzle-Wipe-Routine (in PRINT_START mehrfach genutzt) |
| `M600` | Filament-Change-Trigger |
| `Preheat_TPU` | Vorheiz-Profil für TPU |

**PRINT_START-Ablauf im Detail:** Full Home → bei Bett-Ziel >90 °C automatischer Heatsoak mit Nevermore-Aktivierung und Warten auf Kammertemperatur, sonst 1 Minute Soak → Hotend auf 150 °C für sauberes Z-Homing → Nozzle-Reinigung → QGL + erneutes Z-Homing → Bed-Mesh- und Skew-Profil laden → Hotend auf Zieltemp → erneutes Nozzle-Clean → Druckstart.

---

## 🧩 KAMP (Klipper Adaptive Meshing & Purging)

- Aktiv: `Adaptive_Meshing.cfg`, `Line_Purge.cfg`, `Smart_Park.cfg`
- Deaktiviert: `Voron_Purge.cfg` (Logo-Purge)
- Purge-Menge 30 mm³/s Flow, Purge-Höhe 0.8 mm
- Dockable-Probe-Support ist vorhanden, aber deaktiviert (`probe_dock_enable: False`) – logisch, da Tap fest am Toolhead sitzt

---

## 📡 Moonraker & Update-Manager

`moonraker.conf` verwaltet Auto-Updates für:

- Mainsail (+ mainsail-config)
- KlipperScreen
- Klipper-Adaptive-Meshing-Purging (KAMP)
- Crowsnest, Sonar (WLAN-Keepalive)
- Mobileraker Companion
- Klippain Shake&Tune
- klipper_auto_speed
- klipper-backup (das Backup-System selbst)
- moonraker-obico

Trusted Clients umfassen die üblichen privaten IP-Ranges (10.0.0.0/24, 192.168.0.0/16 etc.) – passt zu deinem Heimnetz-Setup.

---

## 📷 Webcam (Crowsnest)

- `ustreamer`-Modus, `/dev/video0`, 1020×768 @ 15 FPS
- Stream über Port 8080 (`/webcam/?action=stream`)
- RTSP deaktiviert

---

## 📱 Remote-Zugriff & Cloud-Dienste

Es sind gleich **drei** Remote-Access-Lösungen parallel eingebunden:

1. **Mobileraker** – App-Companion, Sprache Deutsch, Zeitzone CEST
2. **Obico** (`moonraker-obico`) – KI-Fehlererkennung/Remote-Monitoring
3. **OctoEverywhere** – Remote-Zugriff über Cloud-Relay, Beta-Channel

> ⚠️ **Sicherheitshinweis:** In `moonraker-obico.cfg` liegt ein Klartext-`auth_token` für den Obico-Cloud-Zugriff. Das gehört **nicht** in ein öffentliches GitHub-Repo. Ich würde empfehlen, diese Datei per `.gitignore` auszuschließen (wie es fürs Klipper-Backup-Projekt auch für `saved_variables.cfg` & Co. üblich ist) und den Token bei Obico neu zu generieren, falls das Repo bereits öffentlich gepusht wurde.

---

## 📁 Datei-Struktur (Includes in `printer.cfg`)

```
printer.cfg
├── shell_command.cfg      (Git-Backup-Trigger-Makro)
├── homing.cfg             (Sensorless Homing X/Y, Homing-Override)
├── mainsail.cfg
├── KAMP_Settings.cfg      (→ KAMP/Adaptive_Meshing, Line_Purge, Smart_Park)
├── macros.cfg             (PRINT_START/END, Pause/Resume, Filament etc.)
├── stealthburner_leds.cfg (LED-Statusanzeigen)
└── moonraker_obico_macros.cfg (Layer-Change-Hook für Obico)
```

---

## 🙏 Credits

Basiert auf dem [Klipper-Backup-Projekt](https://github.com/Staubgeborener/klipper-backup) von Staubgeborener.

