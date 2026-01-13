# ADR 0011: Built-in CircuitPython Libraries in Custom Firmware

- Status: accepted
- Date: 2026-01-13
- CircuitPython version: 10.0.3

## Context

Following the decision in [ADR 0010: Custom CircuitPython Build for XIAO nRF52840 Plus](0010-custom-circuitpython.md), we have a custom firmware build. To
ensure developers know which modules are available for use and where to find their documentation, we need a definitive list of built-in
modules included in this specific build.

This list represents the API surface available on the device after pruning unneeded subsystems like displayio, audio, and others.

## Decision

The following modules are included in the custom CircuitPython (v10.0.3) firmware for XIAO nRF52840 Plus. These are built into the core and do not require additional
files on the `CIRCUITPY` drive.

| Module                | Documentation                                                                                                                             | origin   |
|:----------------------|:------------------------------------------------------------------------------------------------------------------------------------------|:---------|
| `_asyncio`            | [docs.circuitpython.org](https://docs.circuitpython.org/en/latest/shared-bindings/_asyncio/index.html)                                    | built-in |
| `_bleio`              | [_bleio](https://docs.circuitpython.org/en/latest/shared-bindings/_bleio/index.html#module-_bleio)                                        | built-in |
| `adafruit_bus_device` | [adafruit_bus_device](https://docs.circuitpython.org/en/latest/shared-bindings/adafruit_bus_device/index.html#module-adafruit_bus_device) | built-in |
| `aesio`               | [aesio](https://docs.circuitpython.org/en/latest/shared-bindings/aesio/index.html#module-aesio)                                           | built-in |
| `alarm`               | [alarm](https://docs.circuitpython.org/en/latest/shared-bindings/alarm/index.html#module-alarm)                                           | built-in |
| `analogio`            | [analogio](https://docs.circuitpython.org/en/latest/shared-bindings/analogio/index.html#module-analogio)                                  | built-in |
| `array`               | [array](https://docs.circuitpython.org/en/latest/docs/library/array.html#module-array)                                                    | built-in |
| `atexit`              | [atexit](https://docs.circuitpython.org/en/latest/shared-bindings/atexit/index.html#module-atexit)                                        | built-in |
| `binascii`            | [binascii](https://docs.circuitpython.org/en/latest/docs/library/binascii.html#module-binascii)                                           | built-in |
| `bitbangio`           | [bitbangio](https://docs.circuitpython.org/en/latest/shared-bindings/bitbangio/index.html#module-bitbangio)                               | built-in |
| `board`               | [board](https://docs.circuitpython.org/en/latest/shared-bindings/board/index.html#module-board)                                           | built-in |
| `builtins`            | [builtins](https://docs.circuitpython.org/en/latest/docs/library/builtins.html#module-builtins)                                           | built-in |
| `busio`               | [busio](https://docs.circuitpython.org/en/latest/shared-bindings/busio/index.html#module-busio)                                           | built-in |
| `codeop`              | [codeop](https://docs.circuitpython.org/en/latest/shared-bindings/codeop/index.html#module-codeop)                                        | built-in |
| `collections`         | [collections](https://docs.circuitpython.org/en/latest/docs/library/collections.html#module-collections)                                  | built-in |
| `countio`             | [countio](https://docs.circuitpython.org/en/latest/shared-bindings/countio/index.html#module-countio)                                     | built-in |
| `digitalio`           | [digitalio](https://docs.circuitpython.org/en/latest/shared-bindings/digitalio/index.html#module-digitalio)                               | built-in |
| `errno`               | [errno](https://docs.circuitpython.org/en/latest/docs/library/errno.html#module-errno)                                                    | built-in |
| `getpass`             | [getpass](https://docs.circuitpython.org/en/latest/shared-bindings/getpass/index.html#module-getpass)                                     | built-in |
| `io`                  | [io](https://docs.circuitpython.org/en/latest/docs/library/io.html#module-io)                                                             | built-in |
| `json`                | [json](https://docs.circuitpython.org/en/latest/docs/library/json.html#module-json)                                                       | built-in |
| `locale`              | [locale](https://docs.circuitpython.org/en/latest/shared-bindings/locale/index.html#module-locale)                                        | built-in |
| `math`                | [math](https://docs.circuitpython.org/en/latest/shared-bindings/math/index.html#module-math)                                              | built-in |
| `memorymap`           | [memorymap](https://docs.circuitpython.org/en/latest/shared-bindings/memorymap/index.html#module-memorymap)                               | built-in |
| `microcontroller`     | [microcontroller](https://docs.circuitpython.org/en/latest/shared-bindings/microcontroller/index.html#module-microcontroller)             | built-in |
| `msgpack`             | [msgpack](https://docs.circuitpython.org/en/latest/shared-bindings/msgpack/index.html#module-msgpack)                                     | built-in |
| `nvm`                 | [nvm](https://docs.circuitpython.org/en/latest/shared-bindings/nvm/index.html#module-nvm)                                                 | built-in |
| `os`                  | [os](https://docs.circuitpython.org/en/latest/shared-bindings/os/index.html#module-os)                                                    | built-in |
| `pulseio`             | [pulseio](https://docs.circuitpython.org/en/latest/shared-bindings/pulseio/index.html#module-pulseio)                                     | built-in |
| `pwmio`               | [pwmio](https://docs.circuitpython.org/en/latest/shared-bindings/pwmio/index.html#module-pwmio)                                           | built-in |
| `random`              | [random](https://docs.circuitpython.org/en/latest/shared-bindings/random/index.html#module-random)                                        | built-in |
| `re`                  | [re](https://docs.circuitpython.org/en/latest/docs/library/re.html#module-re)                                                             | built-in |
| `rtc`                 | [rtc](https://docs.circuitpython.org/en/latest/shared-bindings/rtc/index.html#module-rtc)                                                 | built-in |
| `select`              | [select](https://docs.circuitpython.org/en/latest/docs/library/select.html#module-select)                                                 | built-in |
| `storage`             | [storage](https://docs.circuitpython.org/en/latest/shared-bindings/storage/index.html#module-storage)                                     | built-in |
| `struct`              | [struct](https://docs.circuitpython.org/en/latest/shared-bindings/struct/index.html#module-struct)                                        | built-in |
| `supervisor`          | [supervisor](https://docs.circuitpython.org/en/latest/shared-bindings/supervisor/index.html#module-supervisor)                            | built-in |
| `sys`                 | [sys](https://docs.circuitpython.org/en/latest/docs/library/sys.html#module-sys)                                                          | built-in |
| `time`                | [time](https://docs.circuitpython.org/en/latest/shared-bindings/time/index.html#module-time)                                              | built-in |
| `traceback`           | [traceback](https://docs.circuitpython.org/en/latest/shared-bindings/traceback/index.html#module-traceback)                               | built-in |
| `usb_cdc`             | [usb_cdc](https://docs.circuitpython.org/en/latest/shared-bindings/usb_cdc/index.html#module-usb_cdc)                                     | built-in |
| `usb_hid`             | [usb_hid](https://docs.circuitpython.org/en/latest/shared-bindings/usb_hid/index.html#module-usb_hid)                                     | built-in |
| `warnings`            | [warnings](https://docs.circuitpython.org/en/latest/shared-bindings/warnings/index.html#module-warnings)                                  | built-in |
| `watchdog`            | [watchdog](https://docs.circuitpython.org/en/latest/shared-bindings/watchdog/index.html#module-watchdog)                                  | built-in |
| `zlib`                | [zlib](https://docs.circuitpython.org/en/latest/shared-bindings/zlib/index.html#module-zlib)                                              | built-in |

## Consequences

- Positive outcomes
    - Clear inventory of available capabilities for developers.
    - Fast access to relevant module documentation via links.
    - Simplified debugging (knowing what is a built-in vs. an external library).
- Trade-offs / risks
    - This list must be updated if the custom build configuration (`mpconfigboard.mk`) changes significantly.
- Follow-ups
    - Any additional library needed for the project that is not on this list must be provided as a `.py` or `.mpy` file in the `lib/` directory of the
      `CIRCUITPY` drive or frozen into the firmware.

## Notes

- Documentation links point to the latest stable CircuitPython documentation.
