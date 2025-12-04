## Clock4 Wiki

- **What it is:** Dual-MCU desk clock built around STM32L476 (“time”) and STM32L010 (“date”) controllers sharing a 16 MB QSPI flash that doubles as USB mass storage.
- **What’s inside:** Three STM32CubeIDE firmware projects (`mk4-time`, `mk4-date`, `mk4-bootloader`), timezone/map generators under `qspi/`, and extensive CAD for cases, stands, and mounting hardware in `cad/`.
- **How it works:** Bootloader in the first 64 KB of the L476 validates CRC-tagged firmware images dropped onto the USB drive (`FWT.BIN`, `FWD.BIN`), reflashes MCUs automatically, and hands off to the main application which mounts the same flash as FAT and drives the displays/GPS.
- **Need to build one?** See [Hardware and Build](Hardware-and-Build.md) for BOM pointers, printing guidance, and assembly order.
- **Need to update or configure?** See [Firmware and Updates](Firmware-and-Updates.md) for build instructions, timezone data workflow, and the drag-and-drop flashing process.
- **Ready to use it daily?** See [Operating the Clock](Operating-the-Clock.md) for configuration tips, countdown behavior, and troubleshooting notes.

- **Quick facts**
  - USB MSC exposes `config.txt`, timezone binary blobs, and firmware images for easy edits.
  - Countdown inputs accept local ISO timestamps (omit `Z`) or UTC (add `Z`); once the target passes the display flips to `T+` and counts upward automatically.
  - CAD directory includes multiple enclosure revisions plus wall, shelf, antenna, and hinge accessories to suit different setups.
