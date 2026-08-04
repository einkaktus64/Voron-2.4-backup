# 🚀 Voron 2.4 – Klipper Config & Backup 💾

> Vollständige Klipper-Konfiguration und automatisierte Sicherungen für meinen **Voron 2.4 CoreXY** (Xol Toolhead & Chaos Labs CNC Tap).

---

## 🖨️ Voron 2.4 Hardware Setup

Dieses Repository enthält die Live-Konfiguration meines **Voron 2.4**. Dank der Integration von Klipper-Backup werden alle Abstimmungen für den Xol Toolhead, den CNC Tap und sämtliche Klipper-Makros automatisch versioniert und sicher auf GitHub gespeichert.

### ⚙️ Hardware & Spezifikationen
* **Kinematik & Leveling:** CoreXY mit Quad Gantry Leveling (QGL) & Bed Mesh
* **Toolhead:** ⚡ **Xol Toolhead** *(leichtgewichtig & optimierter Bauteillüfter-Airflow)*
* **Z-Probe / Homing:** 🎯 **Chaos Labs CNC Tap** *(extrem präzises Homing direkt über die Düse)*
* **Software-Stack:** Klipper + Moonraker
* **Web-Interface:** Mainsail 

---

## 📁 Was wird gesichert?

Hier sind alle wichtigen Steuerungs- und Konfigurationsdateien hinterlegt:

* 📄 `printer.cfg` – Pin-Belegungen, Stepper-Treiber, Kinematik & Limits
* 🎯 `tap.cfg` / `stepper_z.cfg` – Tap-Homing, Z-Offset & Sicherheits-Limits
* 📜 `macros.cfg` – Voron-Makros (`PRINT_START`, `PRINT_END`, Park-Positionen, Heatup)
* 📐 `quad_gantry_level.cfg` / `bed_mesh.cfg` – Nivellierung & Bett-Kompensation
* 📡 `moonraker.conf` & `crowsnest.conf` – API-, Update- und Webcam-Konfiguration

---

## ✨ Backup-Features

* 🔄 **Vollautomatisch:** Änderungen in Mainsail/Fluidd werden direkt nach dem Speichern gepushed.
* 🛡️ **Quick Recovery:** Bei SD-Karten-Defekt oder Neuaufsetzen des Pi sofort einsatzbereit.
* 📜 **Verlauf:** Jede Anpassung (z.B. Z-Offset-Kalibrierung für den Tap oder Input-Shaping) bleibt lückenlos nachvollziehbar.

---

## 🙏 Credits & Werkzeuge

Das automatisierte Backup basiert auf dem Klipper-Backup-Projekt:

[![Klipper-Backup](https://img.shields.io/badge/GitHub-Klipper--Backup-blue?style=for-the-badge&logo=github)](https://github.com/Staubgeborener/klipper-backup)

---
*Powered by Voron Design, Xol Toolhead, Chaos Labs & Klipper*
