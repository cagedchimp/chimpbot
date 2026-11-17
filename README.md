# ChimpBot

Config backup for my Voron 2.4 (350 mm), "ChimpBot 2.4 350".

## Hardware
- Mainboard: BigTreeTech Octopus V1 (STM32F446), TMC2209 drivers
- Toolhead: Stealthburner, Clockwork 2 extruder, LDO Nitehawk-SB toolboard (RP2040, USB)
- Hotend: PT1000 sensor; Voron Tap for Z and QGL
- Bed: 350 mm, SSR, Generic 3950 thermistor
- Extras: Nevermore filter, Decontaminator purge bucket + brush, mini12864 display, KlipperScreen, two USB webcams
- Host: Raspberry Pi 4 (4 GB), Debian 12 bookworm

## Layout
| Path | Contents |
|---|---|
| `config/` | Everything from `~/printer_data/config` that matters: live configs, old printer.cfg variants, Moonraker/Crowsnest/OctoEverywhere/Obico configs |
| `system/machine-info.json` | Software versions, OS, MCU serial paths, print history totals, list of what wasn't backed up |
| `system/mainsail-ui-db.json` | Mainsail UI settings and webcam definitions from the Moonraker database |
| `proposed/` | New macros, not installed yet |
| `IMPROVEMENT_PLAN.md` | Findings, software plan, and hardware upgrades with costs |

## Notes
- Secrets are redacted (`<REDACTED>`), for example the Obico `auth_token`.
- Trailing whitespace was trimmed. Every file was checked against the printer's copy and matches line for line otherwise.
- Not included: the 93 `printer-YYYYMMDD_HHMMSS.cfg` SAVE_CONFIG autosaves, old `config-*.zip` / `logs-*.zip` archives, and gcode files.

Captured 17 Nov 2026.
