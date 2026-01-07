# ADR 0007: MicroPython Soft Breakpoints for Interactive Debugging

- Status: implemented
- Date: 2026-01-07

## Context

We develop firmware for Raspberry Pi Pico W using MicroPython and debug primarily over USB-serial. We need a workflow that supports:

- Pausing execution at selected points, inspecting relevant state, and then continuing.
- Avoiding reliance on `Ctrl-C` and REPL for interactive inspection, because `Ctrl-C` interrupts execution and cannot resume the same stack/frame.
- Lightweight instrumentation with minimal dependencies and low friction for frequent use.

Constraints and realities:

- MicroPython does not provide IDE-style breakpoints/step debugging for Python code.
- Blocking inside IRQ/callback contexts (e.g., BLE IRQ) is unsafe and can stall the stack.
- In `uasyncio`, blocking calls (including `input()`) stall the event loop; this is acceptable only as an intentional diagnostic pause.

## Decision

Adopt a "soft breakpoint" primitive implemented in `debug/soft_break.py`:

- Provide `bp(tag, ...)` that:
    - Maintains a per-`tag` sequence counter (`tag#sequence`) to identify repeated breakpoint hits and enable rate-limiting.
    - Supports an optional `predicate` gating function to decide whether to stop.
        - The sequence counter is incremented first, then `predicate` is evaluated.
        - If `predicate` is false, the function returns without stopping.
        - Predicate calling convention:
            - Prefer `predicate(tag, sequence)` if accepted.
            - Else try `predicate(predicate_arg)` if accepted.
            - Else call `predicate()` (zero-argument).
    - Supports optional `with_log` to emit a one-line breadcrumb log even when not stopping.
    - Captures user-selected debug data in a global `DBG` dictionary (explicit parameters passed via `**kw`).
    - Provides an interactive console prompt over USB-serial to inspect, in priority order:
        - `locals_map` if explicitly passed (`l`)
        - `globals_map` if explicitly passed (`g`)
        - `DBG` snapshot (`p`)
        - Help (`h`)
        - Continue (`Enter`)
    - Uses summarized printing to reduce noise and avoid dumping large or unhelpful values.
        - Filters out modules/types/functions/methods in summaries by default.

In scope:

- Manual debug stop points in normal (non-IRQ) execution context.
- Per-tag sequencing and predicate gating to control breakpoint frequency.
- Minimal interactive commands for quick inspection and continuation.

Out of scope:

- Capturing caller function locals automatically (no `sys._getframe()` on this MicroPython build).
- Safe use within IRQ/callback contexts.
- Step-by-step debugging or watch windows.

## Consequences

- Positive outcomes
    - Enables "pause, inspect, continue" on real hardware without REPL stack disruption.
    - Tag + sequence provides stable identifiers and supports common patterns like "every N-th hit".
    - Predicate gating avoids noisy stops and supports selective triggers (e.g., only on anomalies).
    - Optional breadcrumb logging (`with_log`) supports lightweight tracing even when not breaking.
    - Explicit `locals()` / `globals()` support targeted inspection without requiring runtime stack introspection.

- Trade-offs / risks
    - Breakpoints are blocking by design (`input()`):
        - In `uasyncio`, they freeze the entire event loop while active.
        - They must not run inside IRQ/callback contexts (risk of stalling BLE and distorting behavior).
    - Locals/globals inspection requires explicit passing of `locals()` / `globals()`.
    - Snapshot printing is intentionally summarized; full dumps require manual printing or custom hooks.

- Follow-ups (if any)
    - Establish usage guidelines:
        - Allowed contexts: main loop, `uasyncio` tasks.
        - Disallowed contexts: IRQ/callback handlers (BLE IRQ, timers).
    - Add a deferred-break pattern where IRQ sets a flag and `bp()` is called later from a task/main loop.
    - Standardize tag taxonomy (e.g., `MTU`, `RX`, `TX`, `FSM`, `ERR`) for consistent logs and breakpoint usage.

## Notes

- File
    - `debug/soft_break.py`

- Command set (current, in importance order)
    - `l`: print `locals_map` (if provided)
    - `g`: print `globals_map` (if provided)
    - `p`: print `DBG`
    - `h`: help
    - `Enter`: continue

- Examples

  Minimal breakpoint with snapshot, locals/globals, and breadcrumb logging:

```python
  from debug.soft_break import bp

bp(
    "MTU",
    with_log=True,
    locals_map=locals(),
    globals_map=globals(),
    mtu=mtu,
    att_payload=mtu - 3,
)
```

Every 50th hit per tag (plus breadcrumb logging every time `bp()` is evaluated):

```python
bp(
    "RX",
    predicate=lambda tag, sequence: (sequence % 50) == 0,
    with_log=True,
    locals_map=locals(),
    rx_len=len(rx),
)
```

Break only if a single value meets a condition:

```python
bp(
    "MTU",
    predicate=lambda mtu: mtu > 200,
    predicate_arg=mtu,
    with_log=True,
    mtu=mtu,
)
```

End-to-end example and expected console session:

```python
from debug.soft_break import bp

g_x = 10


def main():
    l_var = 15500
    l_l = [10, 20, 30]
    l_d = {"a": 10, "b": 20}
    s_s = "hello world"

    bp(
        "main tag",
        locals_map=locals(),
        globals_map=globals(),
        with_log=True,
        state="aaa",
        mtu=11,
        rx_len=22,
    )


if __name__ == "__main__":
    main()
```

```plaintext
00:00.001	[    INFO]	APP:	start
00:00.001	[    INFO]	 BP:	bp: main tag#1 @00:00.001
DBG:
  mtu [int]: 11
  rx_len [int]: 22
  state [str]: len=3 'aaa'
bp: main tag#1 @00:00.001> [Enter=continue, l=locals, g=globals, p=DBG, h=help] l
locals:
  l_d [dict]: len=2 keys=['a', 'b']
  l_l [list]: len=3
  l_var [int]: 15500
  s_s [str]: len=11 'hello world'
bp: main tag#1 @00:00.001> [Enter=continue, l=locals, g=globals, p=DBG, h=help] g
globals:
  g_x [int]: 10
bp: main tag#1 @00:00.001> [Enter=continue, l=locals, g=globals, p=DBG, h=help] 
```
