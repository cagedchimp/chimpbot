# Proposed macros (not installed yet)

These files are drafts written against the live config backed up in `../config/`.
They have **not** been loaded on the printer. Install them one step at a time
(see `../IMPROVEMENT_PLAN.md`, Phase 2 and 3), restart Klipper after each step
and watch the console for errors.

| File | Replaces | What it does |
|---|---|---|
| `chimpbot_settings.cfg` | settings scattered across files | `_CHIMP` settings + Mainsail `_CLIENT_VARIABLE` (park spot, 12 h pause timeout, cancel clean-up) |
| `print_start_end.cfg` | `PRINT_START`/`PRINT_END` in printer.cfg, `START_PRINT`/`END_PRINT`/`PURGE`/`G29` in kiauh_macros.cfg | Material-aware start: Nevermore + soak for ABS/ASA, chamber wait if a sensor exists, Tap-safe probing at 150C, adaptive mesh, heat over the bucket, scrub, purge line next to the part |
| `filament.cfg` | `LOAD_FILAMENT`/`UNLOAD_FILAMENT`/`M600` in kiauh_macros.cfg | Material presets, tip-forming unload, M600 that shows a Load / Purge / Resume dialog in Mainsail |
| `quality_of_life.cfg` | lights/park bits | `PREHEAT MATERIAL=ABS`, `COOLDOWN`, `PARK_FRONT`, `CG28`, lights switch off 15 min after a print |

Includes, in this order, after removing `[include kiauh_macros.cfg]`:

```
[include mainsail.cfg]          # from ~/mainsail-config (symlink)
[include chimpbot_settings.cfg]
[include print_start_end.cfg]
[include filament.cfg]
[include quality_of_life.cfg]
```
