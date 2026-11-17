# ChimpBot improvement plan

Written 17 Nov 2026 from the config backed up in this repo. The focus is day-to-day use: fewer surprises, less babysitting, and macros that do the right thing for each material.

## What I found

These are the things in the current config that most affect how the printer feels to use.

1. **Two sets of PAUSE / RESUME / CANCEL macros are fighting.** `printer.cfg` includes `mainsail.cfg` and then `kiauh_macros.cfg`. Klipper merges sections with the same name, so the KIAUH versions win. In practice:
   - Cancel doesn't park the head or run `PRINT_END`. It turns heaters off and stops where it is.
   - Pause parks at X20 Y20 and doesn't change the idle timeout. `idle_timeout` is 30 minutes, so a filament change that takes longer turns off the heaters and the motors, and the print can't be resumed.
   - `M600`, `LOAD_FILAMENT` and `UNLOAD_FILAMENT` also come from `kiauh_macros.cfg`.
2. **`LOAD_FILAMENT` has a bug.** It ends with `G29 E0`, and `G29` is the bed-mesh macro. Loading filament from Mainsail or KlipperScreen therefore heats the bed and runs a full bed mesh. It also only heats to 190 °C, which is too cold for PETG or ABS.
3. **`PRINT_START` does more work than it needs to and skips some useful steps.**
   - It probes a full 5×5 mesh on every print. Klipper has adaptive meshing built in now (`BED_MESH_CALIBRATE ADAPTIVE=1`).
   - The heat soak is commented out, and the Nevermore never turns on by itself.
   - The purge line is always at the front left.
   - `nozzle_scrub.cfg` only purges at 240 °C or hotter, so PLA and PETG prints never purge into the bucket.
   - `PRINT_END` turns the chamber lights off right away, so you come back to a dark printer.
4. **Unused and leftover files.**
   - `Chamber_Lighting.conf` sets the case light on `P2.5`, which is an SKR pin. It only works because `printer.cfg` redefines the light later.
   - `skew_correction.cfg` is Python source code (a Klipper module), not a config file.
   - `filament_runout.cfg` is empty. `macros.cfg`, `M600.cfg`, `filament.cfg`, `simple pause.cfg`, `pause_resume_cancel.cfg` and `LOAD/UNLOAD_FILAMENT.cfg` aren't included anywhere.
   - There are four old `printer.cfg` variants, 93 SAVE_CONFIG autosaves and nine zip archives (about 120 MB). They clutter Mainsail's config list.
5. **The software is 1–2 years behind.**

   | Component | Installed | Available |
   |---|---|---|
   | Klipper | v0.12.0-396 | v0.13.0-822 |
   | Moonraker | v0.9.3 | v0.11.0 |
   | Mainsail | v2.12.0 | v2.19.0 |
   | KlipperScreen | v0.4.4 | v0.4.7 |
   | Crowsnest | v4.1.9 | v4.2.0 |
   | OctoEverywhere | 3.6.3 | 5.4.1 |
   | Obico | 2.0.9 | 2.2.4 |

   `mainsail.cfg` is a static copy from February 2024. It isn't linked to the `mainsail-config` repo, so updates never reach it.
6. **The webcam names are swapped in Mainsail.** "Angry" shows the Logitech stream (`/webcam2`), and "Logi - Top" shows the other camera. "Angry" also has flip X + flip Y + rotate 180°, and those cancel out.
7. **Obico and OctoEverywhere are both running.** You get two remote-access services, two failure detectors and two sets of notifications, all on one Pi 4.
8. **Some hardware that's already there isn't set up.**
   - The Nitehawk-SB has an accelerometer, but it's commented out. Input shaper was last tuned with older hardware (2hump_ei 48.8 / ei 42 Hz), and `max_accel` is a cautious 3000.
   - The Stealthburner LED section is commented out, so the toolhead gives no status at a glance.
9. **The Pi is on Wi-Fi only,** and it stores about 6 GB of gcode on a microSD card.
10. **Mainsail maintenance reminders are empty.** The feature is installed but has no entries.

## Software plan

Do the phases in order. Commit to this repo before and after each phase so you can always roll back.

### Phase 0: Automatic backups (30 min)
- [ ] Install [klipper-backup](https://github.com/Staubgeborener/klipper-backup) on the Pi, pointed at this repo, with a fine-grained GitHub token that can only write `chimpbot`. Set it to run on every config save and nightly.
- [ ] Add a `.gitignore` for `*.zip`, `printer-*.cfg` and anything containing tokens. `moonraker-obico.cfg` has an auth token, which is redacted in this backup.
- [ ] Move the old variants, autosaves and zips on the Pi into `~/printer_data/config/archive/`. Mainsail ignores subfolders you don't include.

### Phase 1: Update everything (1–2 h)
- [ ] Take a full Mainsail backup first (Machine → Backup), in addition to this repo.
- [ ] Update the OS packages, then Moonraker, Klipper, Mainsail, KlipperScreen, Crowsnest and the remote service from the Mainsail update panel.
- [ ] Rebuild and flash firmware for both MCUs. The Klipper 0.12 → 0.13 jump requires it.
  - Octopus V1: STM32F446, 32 KiB bootloader, 12 MHz crystal, USB.
  - Nitehawk-SB: RP2040, follow LDO's update steps.
- [ ] Switch `mainsail.cfg` to the official `mainsail-config` repo (`~/mainsail-config/client.cfg` symlinked as `mainsail.cfg`, plus its `[update_manager mainsail-config]` block), so the pause and cancel macros stay current.
- [ ] Pick one remote service. Keep OctoEverywhere (Gadget AI, app, notifications) or Obico, and uninstall the other. Remove its include from `printer.cfg` and `moonraker.conf`.
- [ ] Fix the webcam names and flips in Mainsail. Settings → Webcams: name each one after the stream it actually shows, and remove the double flip.

### Phase 2: Clean up the macros (1 h)
- [ ] Remove `[include kiauh_macros.cfg]`. Keep `[gcode_arcs]` by moving `resolution: 1.0` into `printer.cfg`.
- [ ] Delete `[include Chamber_Lighting.conf]`. The case light and its macros are already in `printer.cfg`.
- [ ] Add `proposed/chimpbot_settings.cfg`. It sets Mainsail's park spot to front-center, the pause timeout to 12 h, and cancel to park at the back and start the filter and light timers.
- [ ] Add `proposed/filament.cfg`. It brings material presets (`LOAD_FILAMENT MATERIAL=PETG`) and a tip-forming unload. `M600` then opens a dialog in Mainsail with Load / Purge more / Resume buttons.
- [ ] Add `proposed/quality_of_life.cfg`. It adds `PREHEAT MATERIAL=ABS`, `COOLDOWN`, `PARK_FRONT` and `CG28`, and the lights switch off 15 min after a print.
- [ ] Restart and test each macro with nothing on the bed.

### Phase 3: New print start and end (1 h + test prints)
- [ ] Replace `PRINT_START` and `PRINT_END` in `printer.cfg` with `proposed/print_start_end.cfg`. The new start sequence:
  - turns the Nevermore on and soaks 10 min for ABS/ASA/PC/PA (or waits for chamber temperature once a sensor is fitted)
  - probes with the nozzle at 150 °C (Tap-safe) after a wipe
  - runs QGL and then an adaptive mesh over only the printed area
  - heats over the purge bucket so ooze lands in the bucket
  - scrubs the nozzle, then purges a short line next to the first part
- [ ] In `nozzle_scrub.cfg`, set `variable_purge_temp_min: 200` so PLA and PETG also purge into the bucket.
- [ ] Update the slicer start gcode (the exact lines are at the top of `print_start_end.cfg`) and turn on **Label objects**.
- [ ] Run one PLA and one ABS test print and watch the console for each `ChimpBot:` status step.

### Phase 4: Calibration refresh (1–2 h)
- [ ] Enable the Nitehawk's onboard accelerometer (the commented `[adxl345]` and `[resonance_tester]` blocks in `nitehawk-sb.cfg`, probe point 175,175,20). Then run `SHAPER_CALIBRATE` and `SAVE_CONFIG`.
- [ ] Raise `max_accel` to whatever the shaper results recommend for your chosen smoothing, then set `square_corner_velocity` back to 5.
- [ ] Re-run `PID_CALIBRATE` for the hotend and bed after the firmware update, and re-check pressure advance (currently 0.0724).
- [ ] Set Tap's Z offset on a warm bed and save.

### Phase 5: Nice-to-haves in Mainsail and KlipperScreen (1 h)
- [ ] Mainsail maintenance reminders: belts every 200 h, rails every 300 h, nozzle every 500 h, Nevermore carbon every 50 h of ABS.
- [ ] Mainsail macro groups: put Preheat, Load/Unload, Park Front and Cooldown on the dashboard, and hide the internal `_` macros.
- [ ] KlipperScreen: add preheat presets that call `PREHEAT MATERIAL=...` so the touchscreen matches Mainsail.
- [ ] Optional: install moonraker-timelapse, set up Spoolman for spool tracking, and configure Moonraker `[power]` if you buy the smart plug below.

## Hardware recommendations

Prices are rough US street prices as of late 2026. Check current listings before buying.

### Worth doing first (about $60–100 total)
| Item | Why | Est. cost |
|---|---|---|
| Chamber thermistor (100k NTC on the frame, into an Octopus thermistor port) | Real heat soak by temperature instead of a fixed timer; `PRINT_START` already uses it if it's named `temperature_sensor chamber` | $5–10 |
| Filament runout / motion sensor (BTT SFS V2, or a printed switch-type sensor) | Pauses on runout or a jam instead of air-printing; set `runout_sensor` in `_CLIENT_VARIABLE` and Mainsail blocks resume without filament | $5 (DIY) – $25 (SFS V2) |
| Ethernet cable to the Pi | Steadier webcam streams and remote access than Wi-Fi | $0–10 |
| Smart plug that Moonraker can control (Shelly or Tasmota) | Turn the printer on remotely, auto power-off after cool-down | $20–30 |
| Spare Nevermore carbon | Fresh carbon matters more than fan speed | $15–25 |

### Bigger upgrades (pick based on what bugs you)
| Item | Why | Est. cost |
|---|---|---|
| Eddy-current probe (BTT Eddy / Cartographer / Beacon) | Meshes in seconds instead of a minute or more of Tap probing; Beacon and Cartographer can also do contact Z | $30–130 depending on model |
| USB SSD for the Pi (boot from SSD) | microSD wear is the most common Klipper failure; also faster uploads and thumbnails | $25–40 |
| Stealthburner LEDs (if they aren't fitted) + status macros | See heating, leveling and printing state from across the room | $5–15 |
| Fresh Gates belts + a tension gauge | 1,200 printing hours on the current belts; consistent tension helps input shaper | $25–45 |

### Probably not worth it right now
- A Pi 5. The Pi 4 4 GB handles Klipper, two webcams and one remote service fine once the second service is removed.
- A new mainboard. The Octopus V1 with TMC2209s has spare ports for the chamber sensor and a runout sensor.
