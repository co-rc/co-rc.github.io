# ADR 0008: MicroPython Console Logging via `logging` + Timestamped Handler

- Status: implemented
- Date: 2026-01-07

## Context

We need consistent, readable logging over USB-serial for MicroPython firmware on Raspberry Pi Pico W. Requirements:

- Logs printed to the USB console (stdout).
- Simple, stable formatting to aid debugging and correlation with events.
- Timestamp based on monotonic time since start (absolute wall clock not required).
- Leverage built-in `logging` (confirmed available in our MicroPython build) rather than a custom logging framework.

Constraints and realities:

- Console I/O is relatively slow; excessive logging can perturb timing.
- Formatting and advanced `logging` features vary by MicroPython port/build; avoid reliance on rarely-implemented features.
- Timestamp generation is centralized in `debug.time_utils.format_current_stamp()` for consistent formatting across logging and debugging tools.

## Decision

Adopt a `logging`-based console logger implemented in `debug/debug_logging.py`:

- Configure the root logger:
    - Root level set to `INFO` by default.
    - Remove any existing handlers to avoid duplicated output.
    - Attach a custom handler (`_TicksHandler`) that writes to stdout.

- Formatting:
    - Use a `logging.Formatter` with fixed, tab-separated fields:
        - `%(ts)s\t[%(levelname)8s]\t%(name)s:\t%(message)s`
    - `%(ts)s` is injected per record in the handler using `debug.time_utils.format_current_stamp()`.

- Logger names:
    - Use short, stable 3-letter names for subsystem separation:
        - `APP` (application lifecycle / top-level flow)
        - `BLE` (BLE state and events)
        - `PRT` (protocol framing/parsing)

In scope:

- Console-only logging to stdout.
- Standardized format and subsystem logger names.
- Central timestamp formatting via `format_current_stamp()`.

Out of scope:

- Persistent logging (files/flash).
- Remote logging transport.
- Structured/JSON logging.
- Automatic source location (`file:line`) enrichment (not reliably available on this MicroPython build).

## Consequences

- Positive outcomes
    - Consistent, grep-friendly log format across the firmware.
    - Minimal code footprint leveraging built-in `logging`.
    - Timestamping is uniform and reusable by other debug utilities (`soft_break`).

- Trade-offs / risks
    - Removing root handlers is global and may interfere with third-party modules that expect existing handlers.
    - Formatting is intentionally simple; richer metadata requires explicit additions.
    - High-volume logging may impact timing and performance.

- Follow-ups (if any)
    - Define a project-wide policy for log levels (e.g., normal `INFO`, temporary `DEBUG` during investigations).
    - Consider making handler installation idempotent (guard against multiple imports) if needed.
    - Optionally add an `IRQ` logger, with guidance to keep IRQ logs minimal.

## Notes

- Output format example
  `01:23.456 [ INFO] BLE: connected handle=64`

- Example usage
  - ```python
    from debug.debug_logging import APP, BLE, PRT
  
    APP.info("start")
    BLE.info("connected handle=%d", handle)
    PRT.debug("rx frame len=%d", frame_len)
    ```
