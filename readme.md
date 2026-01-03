# Precision Clock Mk IV

Source code for the [Precision Clock Mk IV](https://mitxela.com/projects/precision_clock_mk_iv)

There are three software projects, compiled with STM32CubeIDE:

- mk4-time is the main clock source code running on the STM32L476
- mk4-date is the secondary display running on the STM32L010
- mk4-bootloader runs on the STM32L476 to do firmware updates

In the QSPI folder, there are scripts to create firmware images with CRC that the clock will recognise, along with the scripts to create the tzrules and create a valid disk image, see [these notes](qspi/qspi.md) for more info.

Timezone detection is ported from [ZoneDetect](https://github.com/BertoldVdb/ZoneDetect) by Bertold Van den Bergh and uses shapefile data from [Timezone Boundary Builder](https://github.com/evansiroky/timezone-boundary-builder) by Evan Siroky.

## Configuration (`CONFIG.TXT`)

The external QSPI flash is exposed as a USB mass storage device and is expected to contain a `CONFIG.TXT` at the filesystem root (alongside `TZRULES.BIN` / `TZMAP.BIN`). See `qspi/qspi.md` and the example `qspi/config.txt`.

The config parser is **case-insensitive** for keys and accepts the following boolean-ish values:

- **truthy**: `on`, `enabled`, `1`
- **falsey**: `off`, `disabled`, `0`, `none`

## Predator-style countdown

There are two countdown modes:

- **`MODE_COUNTDOWN`**: normal countdown (decimal HH:MM:SS on the main display; decimal days on the date display)
- **`MODE_PREDATOR_COUNTDOWN`**: “Predator” countdown (hex HH:MM:SS on the main display; hex days on the date display)

Both use `COUNTDOWN_TO` to set the target timestamp. If the target is in the past, the countdown **counts up** instead.

In predator mode, the date display uses a `p-` / `p+` prefix and shows the day count as a **zero-padded hex value**.

Example:

```ini
MODE_PREDATOR_COUNTDOWN = enabled
COUNTDOWN_TO = 2026-01-01T00:00:00Z
```

### `COUNTDOWN_TO` timestamp formats

`COUNTDOWN_TO` accepts:

- **Date only**: `YYYY-MM-DD` (interpreted as midnight)
- **Date + time**: `YYYY-MM-DDTHH:MM` or `YYYY-MM-DDTHH:MM:SS`
- **UTC**: add a trailing `Z` (e.g. `2026-01-01T00:00:00Z`)
- **Explicit offset**: `+HHMM`, `-HHMM`, `+HH:MM`, `-HH:MM` (e.g. `2026-01-01T00:00:00-05:00`)

If you do **not** include `Z` or an explicit offset, the value is treated as **local time in the clock’s current timezone** (including DST rules).
