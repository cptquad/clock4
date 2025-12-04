## Firmware and Updates

### Project Layout
- `mk4-time/` – main STM32L476 application. Handles GNSS parsing, RTC, LED driving, ambient light control, FATFS/USB MSC, and UART link to the date MCU. Includes countdown, countdown-to-count-up logic, and boot coordination.
- `mk4-date/` – STM32L010 firmware that renders text on the date display, responds to UART commands, debounces buttons, and can jump into ROM bootloader on command.
- `mk4-bootloader/` – resident updater occupying the first 64 KB of the L476 flash. Validates CRC-tagged firmware files from QSPI, shows progress animations, erases/programs internal flash, then chains into the main app.
- `qspi/` – tools and docs for the external flash contents: timezone boundary data, transition rule generator (`generate-tzrules.py`), CRC helper (`fw-crc.sh`), release scripts, and format instructions.

### Building Firmware
1. Open each STM32CubeIDE project (or use CLI `make` from within the generated environment) targeting the correct MCU variant.
2. Build `mk4-time` (Release). Copy or symlink the resulting binary to `qspi/output/fwt.bin`.
3. Build `mk4-date`. Copy the output to `qspi/output/fwd.bin`.
4. If the bootloader changes, flash `mk4-bootloader` to the first 64 KB of the L476, then immediately flash the latest `fwt.bin` after it.
5. Run `qspi/fw-crc.sh fwt.bin` and `fw-crc.sh fwd.bin` to append the CRC32 word expected by the bootloaders. Files must remain fixed-size: 192 KB for time, 32 KB for date.

### Timezone and Map Data
- `tzrules.bin` contains precomputed offsets/transitions up to 2100. Generate it by downloading the latest `timezone-names.json` from `timezone-boundary-builder`, ensuring system `tzdata` is current, then running `generate-tzrules.py`.
- `tzmap.bin` is a ZoneDetect-format shapefile export describing timezone polygons. Follow `qspi/qspi.md` to rebuild it via the ZoneDetect database builder.
- Store both files in the root of the QSPI FAT volume (accessible over USB). The time firmware reloads them as needed.

### Config File (`config.txt`)
- Plain text `key=value` pairs. Relevant keys:
  - `countdown_to=YYYY-MM-DDThh:mm[:ss][Z]` – local time if `Z` omitted, UTC if appended; once the event passes, the clock automatically counts upward and shows `T+`.
  - `MODE_*` keys enable/disable display modes (at least one must stay on).
  - `zone_override=Region/City` forces a timezone when GNSS data is missing.
  - `brightness`, `BS1`… configure LED intensity manually or via lookup curve.
  - `nmea=rmc/all/none` controls how much GNSS data is exposed over USB CDC.
- After editing `config.txt` on the mounted drive, eject to trigger a reload.

### Updating Firmware via USB (No STLink Needed)
1. Connect the clock to a computer; it appears as a 16 MB drive. Keep it formatted FAT12/FAT16 with 4096-byte sectors (`qspi/qspi.md` has formatting commands).
2. Copy new `FWT.BIN` (time side) and/or `FWD.BIN` (date side) to the drive. Ensure each file still contains the appended CRC word.
3. Safely eject/unmount the drive. The L476 firmware detects the eject event, compares CRCs, and reboots into the bootloader if needed.
4. The bootloader verifies `FWT.BIN`, erases/programs the L476 application area, and displays progress on the LED columns. When finished, it launches the new app.
5. On startup, `mk4-time` checks `FWD.BIN`. If the CRC differs from the date MCU, it instructs the L010 to enter its ROM bootloader, streams the new firmware over UART, and shows another progress bar.
6. Leave the firmware files on the drive or delete them; future updates only run when CRCs differ.

### Recovery & Troubleshooting
- **CRC errors:** Bootloader shows “Err b###” codes. Recompute CRC via `fw-crc.sh` and recopy the file.
- **Stuck countdown:** Provide a valid `countdown_to` timestamp; setting it to `0` or removing the key hides the mode.
- **No GNSS lock:** Use `MODE_SATVIEW` to inspect satellite counts. Reposition antenna or check `fake_longitude/fake_latitude` overrides for testing.
- **Full wipe:** Reformat the QSPI drive (FAT12/16, 4 KB sectors), recopy `config.txt`, `tzrules.bin`, `tzmap.bin`, and firmware files, then eject to trigger a full reload.
