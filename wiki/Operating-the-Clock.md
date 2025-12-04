## Operating the Clock

### Getting Started
- Plug the clock into USB. It powers the hardware, mounts the QSPI flash as a 16 MB drive, and shows the bootloader animation. First boot may take longer while GNSS lock establishes and timezone data loads.
- Edit `config.txt` on the mounted drive to enable preferred modes, set countdown targets, or override brightness. Eject to apply changes.
- Position the GNSS antenna with a clear sky view; use the “Satview” mode to confirm satellites per constellation. If GPS is unavailable (indoors/lab work), use `fake_longitude`/`fake_latitude` to test.

### Display Modes & Controls
- Buttons advance or reverse through the enabled display modes: ISO8601, ordinal date, ISO week, UNIX seconds, Julian dates, timezone offset/name, weekday, satellite counts, countdown, countdown diagnostics, VBAT, text messages, firmware CRCs, and standby.
- Precision dots (right-hand digits) indicate timing quality. In countdown mode, they switch between countdown and count-up precision tables automatically.
- Text mode displays up to 30 characters of `text=` from the config file; use it for status notes or test messages.

### Countdown & Count-Up
- Configure `countdown_to=YYYY-MM-DDThh:mm[:ss]` for local time or append `Z` for UTC. The firmware runs the timestamp through the active timezone rules, so daylight-saving offsets apply automatically.
- While the target is in the future, the display shows `T-` on the date side with remaining days. Once the target passes, it seamlessly flips to counting upward with `T+`, using the same HH:MM:SS precision stack.
- Setting `countdown_to=0` (or omitting the key) hides the mode; the firmware blanks the digits rather than showing stale values.

### USB Mass Storage Workflow
- After boot, the clock exposes the QSPI flash over USB. Copy `config.txt`, `tzrules.bin`, `tzmap.bin`, `FWT.BIN`, and `FWD.BIN` directly. Finish with a safe eject to trigger firmware reloads or updates.
- If the drive becomes corrupted, reformat as FAT12/16 with 4096-byte sectors (see `qspi/qspi.md`) and recopy the required files.

### Maintenance Tips
- Periodically regenerate timezone data when `tzdata` or `timezone-boundary-builder` releases updates. Copy new binaries to the drive and eject to load them.
- Keep firmware binaries with CRC words appended using `qspi/fw-crc.sh`; otherwise the bootloader will reject them and display an error.
- Clean the acrylic/light pipes gently; keep dust off the ambient light sensor near the phototransistor housing.
- Use the wall/shelf accessories (`cad/wallhanger*`, `shelfstand`) to reroute cables and protect the USB connector from strain.
