# 🚀 Voron 2.4 – Klipper Config & Backup 💾

> Vollständige Klipper-Konfiguration und automatisierte Sicherungen für meinen **Voron 2.4 CoreXY 3D-Drucker**.

---

## 🖨️ Voron 2.4 Setup & Repository-Zweck

Dieses Repository enthält die Live-Konfiguration meines **Voron 2.4**. Dank der Integration von Klipper-Backup werden alle Feinabstimmungen an den Steppern, PID-Werten, Z-Offset-Einstellungen und Makros automatisch versioniert und sicher auf GitHub gespeichert.

### ⚙️ System & Feature Übersicht
* **Firmware:** Klipper + Moonraker
* **Web-Interface:** Mainsail / Fluidd
* **Kinematik & Nivellierung:** CoreXY mit **Quad Gantry Leveling (QGL)** & Bed Mesh
* **Toolhead & Extras:** Stealthburner, Input Shaper & Custom Start-/End-Makros

---

## 📁 Was wird gesichert?

Hier sind alle wichtigen Steuerungs- und Konfigurationsdateien des Voron 2.4 hinterlegt:

* 📄 `printer.cfg` – Pin-Belegungen, Stepper-Treiber, Limits & Kinematik
* 📜 `macros.cfg` – Voron-Makros wie `PRINT_START`, `PRINT_END`, Nozzle Clean & Park-Positionen
* 🎯 `quad_gantry_level.cfg` / `bed_mesh.cfg` – Nivellierung & Bett-Kompensation
* 📡 `moonraker.conf` & `crowsnest.conf` – API-, Update- und Webcam-Kon
