## Hardware and Build

### Core Electronics
- **MCUs:** STM32L476RG (time controller) and STM32L010C6 (date/display driver). The L476 hosts the custom bootloader, GNSS receiver, RTC, QSPI, USB MSC, and UART bridge to the L010.
- **Storage:** 16 MB QSPI NOR flash wired to the L476. Formatted FAT12/16 with 4 KB sectors so it can appear as USB mass storage while also hosting firmware/config/timezone blobs.
- **Peripherals:** GNSS module with PPS output, ambient light sensor (feeds DAC-based dimming), dual LED matrices (time + date), buttons for mode navigation, battery-backed RTC domains, and UART/USB connectors.

### Mechanical Parts
- `cad/main-3dprints-RevC` and `RevD` contain the primary enclosure pieces, diffuser, switch cover, and phototransistor mounts. Choose the revision that matches your PCB layout.
- Additional directories provide accessories: antenna housings, hinges, shelf stands, wall hangers (standard, slim, tape-backed), and measurement aides. All parts ship as STL plus editable STEP/FreeCAD files.
- Recommended materials: PETG or ABS for heat tolerance, translucent PETG/acrylic for light pipes, TPU only where flexible retention is needed.

### Assembly Outline
1. **Print parts:** Slice enclosure pieces at 0.2 mm layers with ≥4 perimeters for stiffness. Print accessory parts as needed (antenna case, hinges, stands).
2. **Populate PCBs:** Solder MCUs, flash, drivers, and connectors. Route PPS to PC7 (EXTI7), UART2 between MCUs, and QSPI lines to the external flash footprint.
3. **Bench test:** Power from a bench supply, flash bootloader + firmware once, verify PPS tick, USB enumeration, and LED scanning before final assembly.
4. **Case integration:** Mount the main board using stand-offs, install diffuser and LED matrices, route GPS antenna through the printed housing, and secure covers (see `Switchcover.step`, `lightsensor-smoked.step`).
5. **Cable management:** For wall/shelf builds, run USB through the hinge/wallhanger channels; secure with printed locks (`wallhanger-slim-lock.stl`, etc.).
6. **Final checks:** Power via USB-C/Micro-B, watch the bootloader animation, confirm the QSPI drive mounts on your PC, and verify GNSS lock (satellite count mode).

### Tools & Supplies
- 3D printer, flush cutters, soldering station/hot air, tweezers, flux, magnification, digital multimeter.
- Optional STLink for low-level recovery (routine updates only need the USB drive).
- Adhesives or hardware per case design (M2/M3 screws, brass inserts, double-sided tape for slim wall mounts).
