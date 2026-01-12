# ADR 0010: Custom CircuitPython Build for XIAO nRF52840 Plus

- Status: accepted
- Date: 2026-01-12

## Context

The firmware target is Seeed XIAO nRF52840 Plus. The device is primarily a BLE-enabled embedded endpoint with optional IR capabilities. Default CircuitPython
builds include many modules that are irrelevant for this device profile (graphics/display stacks, audio stacks, image codecs, NeoPixel/pixel pipeline, and
unused USB classes). These increase firmware size and maintenance surface, and may constrain future additions due to the limited internal flash budget for
firmware (notably frozen modules and additional application libraries compiled to `.mpy` and embedded into the firmware).

Constraints and priorities:

- Keep BLE support.
- USB must support:
    - REPL over USB (CDC)
    - Mass Storage (CIRCUITPY drive)
    - USB HID is kept for roadmap (device may emulate a keyboard for a PC, but the device itself is not a keypad/keyboard matrix).
- No graphics, displays, font rendering, or image decoding.
- No audio features.
- IR is optional/uncertain but kept as a latent capability (do not optimize it away prematurely).
- No planned I2C/SPI sensor peripherals; however, busio may remain enabled unless/ until further size pressure exists.
- Prefer minimal divergence from upstream board definitions; avoid altering board identity/header definitions (VID/PID, flash layout) in the customization
  layer.

## Decision

Adopt a custom CircuitPython build profile for XIAO nRF52840 Plus by maintaining a board configuration override that primarily disables unneeded subsystems,
explicitly trading unused built-in functionality for firmware flash headroom that can be used later for freezing custom libraries and application code.

In scope:

- Maintain an `mpconfigboard.mk` override that:
    - Preserves upstream board header and flash/USB identity configuration.
    - Disables irrelevant, high-footprint module groups:
        - Display/graphics stack (displayio and related buses/drivers/fonts/bitmap tooling)
        - Audio stack (audiocore and related modules)
        - Image codecs (gifio, jpegio)
        - Pixel/NeoPixel stack (pixelmap/pixelbuf/neopixel_write/rainbowio)
        - Numeric stack (ulab)
        - USB MIDI class
    - Keeps USB CDC + MSC + HID, BLE, and general-purpose runtime modules.
- Use the freed firmware flash budget primarily to enable:
    - Frozen modules (`FROZEN_MPY_DIRS`) for custom project libraries and dependencies.
    - Future functional additions without changing the hardware target.

Out of scope:

- Reworking BLE stack configuration or power optimization strategy.
- Converting functionality to C modules or changing the application architecture.
- Aggressively disabling "small" utility modules (e.g., `re`, `zlib`, `msgpack`) unless future size pressure requires it.
- Removing IR-related primitives (`pwmio`, `pulseio`) at this stage; this will be revisited only if/when flash pressure is observed.

Implementation approach:

- Prefer “override by disabling” (`CIRCUITPY_* = 0`) over re-specifying all enabled features.
- If a required feature is not enabled by the upstream board defaults, explicitly enable it (`CIRCUITPY_* = 1`) rather than relying on implicit behavior.

## Consequences

- Positive outcomes
    - Significant reduction of firmware flash usage by removing large unused subsystems (display/audio/pixel/codecs/ulab/usb_midi).
    - Substantial additional headroom in the internal firmware region, enabling larger sets of project-specific frozen libraries and `.mpy`-compiled code.
    - Reduced API surface and fewer accidental dependencies on irrelevant modules.
    - Keeps roadmap flexibility for USB HID while maintaining REPL + CIRCUITPY drive ergonomics.

- Trade-offs / risks
    - Ongoing maintenance cost: board-specific build configuration must be carried forward across CircuitPython updates.
    - Potential dependency surprises: some third-party libraries may import modules that are no longer present.
    - Defaults are not universal: relying on upstream defaults for `= 1` settings is safe only when the base board configuration is known and stable.
    - If later flash pressure appears, the next “big levers” are HID and IR capability; cutting them would reduce roadmap flexibility.

- Follow-ups (if any)
    - Track firmware size and verify critical capabilities after each upstream update (BLE, USB CDC, MSC, HID).
    - Treat freed flash as a budget for frozen project libraries:
        - Prefer freezing stable dependencies to reduce filesystem churn and improve deploy ergonomics.
        - Keep the CIRCUITPY filesystem for configuration and frequently iterated code, not for large dependency payloads.
    - If/when additional size pressure occurs:
        - Reassess keeping IR primitives (`pwmio`, `pulseio`) if IR remains unused.
        - Consider disabling additional utility modules (`re`, `zlib`, `msgpack`, `aesio`) based on actual usage.
    - Maintain a minimal smoke-test checklist for the custom build:
        - USB CDC REPL available
        - CIRCUITPY drive mounts
        - HID enumerates (if enabled)
        - BLE advertising/connectivity works
        - Expected modules are absent (e.g., `displayio`, `audiocore`, `ulab`, `usb_midi`)

## Notes

- This ADR intentionally avoids specifying VID/PID, flash layout, or other board identity headers in the customization layer; these remain sourced from the
  upstream board definition to minimize divergence.
- The customization is designed to be reversible and incremental: additional modules may be disabled later only when justified by measured constraints (size,
  stability, or roadmap changes).

## Custom `mpconfigboard.mk` for XIAO nRF52840 Plus

```text
CIRCUITPY_USB_MIDI = 0

CIRCUITPY_ULAB = 0

CIRCUITPY_AUDIOBUSIO = 0
CIRCUITPY_AUDIOCORE = 0
CIRCUITPY_AUDIOMIXER = 0
CIRCUITPY_AUDIOMP3 = 0
CIRCUITPY_AUDIOPWMIO = 0
CIRCUITPY_SYNTHIO = 0

CIRCUITPY_DISPLAYIO = 0
CIRCUITPY_BUSDISPLAY = 0
CIRCUITPY_FRAMEBUFFERIO = 0
CIRCUITPY_VECTORIO = 0
CIRCUITPY_TERMINALIO = 0
CIRCUITPY_FONTIO = 0
CIRCUITPY_LVFONTIO = 0
CIRCUITPY_BITMAPTOOLS = 0
CIRCUITPY_BITMAPFILTER = 0
CIRCUITPY_TILEPALETTEMAPPER = 0
CIRCUITPY_I2CDISPLAYBUS = 0
CIRCUITPY_FOURWIRE = 0
CIRCUITPY_PARALLELDISPLAYBUS = 0
CIRCUITPY_EPAPERDISPLAY = 0
CIRCUITPY_SHARPDISPLAY = 0
CIRCUITPY_RGBMATRIX = 0

CIRCUITPY_GIFIO = 0
CIRCUITPY_JPEGIO = 0

CIRCUITPY_PIXELMAP = 0
CIRCUITPY_PIXELBUF = 0
CIRCUITPY_NEOPIXEL_WRITE = 0
CIRCUITPY_RAINBOWIO = 0

CIRCUITPY_SDCARDIO = 0
CIRCUITPY_ONEWIREIO = 0
CIRCUITPY_TOUCHIO = 0
CIRCUITPY_ROTARYIO = 0
CIRCUITPY_KEYPAD = 0
CIRCUITPY_KEYPAD_DEMUX = 0
```

## Measuring gains (original vs. custom firmware)

Run after a fresh reset (CTRL-D) on both builds. For a fair comparison, use the same import set and run `gc.collect()` before reading heap metrics.

### REPL report:

```python
import gc, os, sys, microcontroller, supervisor

print("CP", sys.implementation)
print("UID", getattr(microcontroller, "cpu", None) and microcontroller.cpu.uid)

gc.collect()
print("HEAP free/alloc", gc.mem_free(), gc.mem_alloc())

st = os.statvfs("/")
bsize = st[0]
total = bsize * st[2]
free = bsize * st[3]
avail = bsize * st[4]
print("FLASH_FS total/free/avail", total, free, avail)
```

### Optional: verify module presence/absence:

```python
mods = ("_bleio", "usb_cdc", "usb_hid", "usb_midi", "displayio", "audiocore", "ulab", "gifio", "jpegio", "neopixel_write")
for m in mods:
    try:
        __import__(m)
        print(m, "OK")
    except Exception as e:
        print(m, "NO", type(e).__name__, e)
```

### Firmware size (flash footprint):

* Compare the `.uf2` file size on the host machine; this is the most direct indicator of flash savings from removing module groups.

## Result

`make -j24 BOARD=Seeed_XIAO_nRF52840_Sense`

### Original build:

| Memory region                | Used Size                                     | Region Size           | %age Used    |
|------------------------------|-----------------------------------------------|-----------------------|--------------|
| FLASH:                       | 0 B                                           | 1 MB                  | 0.00%        |
| FLASH_MBR:                   | 0 B                                           | 4 KB                  | 0.00%        |
| FLASH_SD:                    | 0 B                                           | 152 KB                | 0.00%        |
| FLASH_ISR:                   | 248 B                                         | 4 KB                  | 6.05%        |
| FLASH_FIRMWARE:              | 605832 B                                      | 776 KB                | 76.24%       |
| FLASH_BLE_CONFIG:            | 0 B                                           | 32 KB                 | 0.00%        |
| FLASH_NVM:                   | 0 B                                           | 8 KB                  | 0.00%        |
| FLASH_FATFS:                 | 0 B                                           | 0 B                   |              |
| FLASH_BOOTLOADER:            | 0 B                                           | 40 KB                 | 0.00%        |
| FLASH_BOOTLOADER_SETTINGS:   | 0 B                                           | 4 KB                  | 0.00%        |
| RAM:                         | 0 B                                           | 256 KB                | 0.00%        |
| SD_RAM:                      | 0 B                                           | 56 KB                 | 0.00%        |
| SPIM3_RAM:                   | 0 B                                           | 8 KB                  | 0.00%        |
| APP_RAM:                     | 47892 B                                       | 192 KB                | 24.36%       |
| ---------------------------- | --------------------------------------------- | --------------------- | ------------ |
| 606080 bytes used,           | 188544 bytes free in flash firmware space     | out of 794624 bytes   | (776.0kB).   |
| 47888 bytes used,            | 214256 bytes free in ram for stack and heap   | out of 262144 bytes   | (256.0kB).   |

filesize(`XIAO_nRF52840_full.uf2`): 1.212.416 bytes

```text
CP (name='circuitpython', version=(10, 0, 3, ''), _machine='Seeed XIAO nRF52840 Sense with nRF52840', _mpy=774, _build='Seeed_XIAO_nRF52840_Sense')
UID bytearray(b'\x80\x08\x13\t\xdcn&\x98')
HEAP free/alloc 135360 3584
FLASH_FS total/free/avail 2072576 2063360 2063360
```

### Custom build:

| Memory region                | Used Size                                     | Region Size           | %age Used    |
|------------------------------|-----------------------------------------------|-----------------------|--------------|
| FLASH:                       | 0 B                                           | 1 MB                  | 0.00%        |
| FLASH_MBR:                   | 0 B                                           | 4 KB                  | 0.00%        |
| FLASH_SD:                    | 0 B                                           | 152 KB                | 0.00%        |
| FLASH_ISR:                   | 248 B                                         | 4 KB                  | 6.05%        |
| FLASH_FIRMWARE:              | 324792 B                                      | 776 KB                | 40.87%       |
| FLASH_BLE_CONFIG:            | 0 B                                           | 32 KB                 | 0.00%        |
| FLASH_NVM:                   | 0 B                                           | 8 KB                  | 0.00%        |
| FLASH_FATFS:                 | 0 B                                           | 0 B                   |              |
| FLASH_BOOTLOADER:            | 0 B                                           | 40 KB                 | 0.00%        |
| FLASH_BOOTLOADER_SETTINGS:   | 0 B                                           | 4 KB                  | 0.00%        |
| RAM:                         | 0 B                                           | 256 KB                | 0.00%        |
| SD_RAM:                      | 0 B                                           | 56 KB                 | 0.00%        |
| SPIM3_RAM:                   | 0 B                                           | 8 KB                  | 0.00%        |
| APP_RAM:                     | 45636 B                                       | 192 KB                | 23.21%       |
| ---------------------------- | --------------------------------------------- | --------------------- | ------------ |
| 325040 bytes used,           | 469584 bytes free in flash firmware space     | out of 794624 bytes   | (776.0kB).   |   
| 45632 bytes used,            | 216512 bytes free in ram for stack and heap   | out of 262144 bytes   | (256.0kB).   |  

filesize(`XIAO_nRF52840_custom.uf2`): 650.240 bytes

```text
CP (name='circuitpython', version=(10, 0, 3, ''), _machine='Seeed XIAO nRF52840 Sense with nRF52840', _mpy=774, _build='Seeed_XIAO_nRF52840_Sense')
UID bytearray(b'\x80\x08\x13\t\xdcn&\x98')
HEAP free/alloc 135648 3360
FLASH_FS total/free/avail 2072576 2063360 2063360
```

## Summary

- Measurement summary (original vs. custom build)

The primary gain is internal firmware flash headroom (for freezing additional project libraries and `.mpy`-compiled code), while the CIRCUITPY filesystem size
remains unchanged (QSPI flash filesystem).

- Firmware flash (FLASH_FIRMWARE region)
  - Used: 606,080 B → 325,040 B (−281,040 B, ~−274.5 KiB, ~−46.4%)
  - Free: 188,544 B → 469,584 B (+281,040 B)
- UF2 artifact size (host)
  - 1,212,416 B → 650,240 B (−562,176 B, ~−549.0 KiB, ~−46.4%)
- RAM (APP_RAM and heap, post-`gc.collect()`)
  - APP_RAM used: 47,888 B → 45,632 B (−2,256 B, ~−2.2 KiB)
  - HEAP free/alloc: 135,360/3,584 → 135,648/3,360 (free +288 B, alloc −224 B)
- Filesystem (CIRCUITPY on QSPI)
  - total/free/avail unchanged: 2,072,576 / 2,063,360 / 2,063,360 bytes
- Module removal verification (custom build)
  - Present: `_bleio`, `usb_cdc`, `usb_hid`
  - Absent (ImportError): `usb_midi`, `displayio`, `audiocore`, `ulab`, `gifio`, `jpegio`, `neopixel_write`
