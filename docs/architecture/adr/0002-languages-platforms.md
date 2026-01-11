# 0002: Platforms and Languages

- Status: accepted
- Date: 2026-01-10

## Context

The project requires a coherent and stable technology stack spanning embedded devices and a mobile frontend. Early architectural decisions should minimize
future migration costs, especially with respect to Bluetooth Low Energy (BLE) behavior and power-management capabilities.

The stack is intentionally constrained to technologies that can be realistically developed, tested, and maintained by a single developer.

## Decision

### Team

- Single-developer project.
- External contributions are possible but not actively planned.

### Platforms

#### Microcontrollers

- **nRF52840**: Primary platform.
    - Chosen for native BLE support and low-power capabilities.
    - Intended to remain the long-term hardware target, including future sleep-enabled operation.
- **Raspberry Pi Pico W**: Secondary / experimental platform.
    - Useful for active, always-on scenarios and early prototyping.
    - Not suitable for low-power BLE operation; therefore not a long-term target.
- **ESP32**: Explicitly not selected.
    - Technically capable, but excluded due to ecosystem complexity and stability concerns.
- **Raspberry Pi (full OS)**: Out of scope.

#### Mobile

- **Android**: Primary and only supported mobile platform.
    - Target: minSdk 36.
    - No backward compatibility.
- iOS: Not planned (no testing hardware available).

#### Desktop

- Windows: Possible future tooling or support utilities.
- Linux: Not planned.

### Languages

#### Microcontrollers

- **Python**: Primary language.
    - **CircuitPython**: Primary runtime on nRF52840.
        - Chosen for BLE stability, high-level APIs, and predictable behavior when introducing sleep.
    - **MicroPython**: Secondary option.
        - Used where lower-level control or tighter timing is required (e.g. RP2040-based boards).
- **C / C++**: Fallback option.
    - Reserved for cases where Python proves insufficient due to performance, power consumption, or BLE stack limitations.
    - Not the default choice.

#### Mobile

- **Java (Android)**: Java 21.
- Kotlin: Not planned.

### IDEs

- Android / Java: IntelliJ Ultimate with Android plugin.
- Microcontrollers / Python: PyCharm with CircuitPython / MicroPython tooling.

## Consequences

1. The technology stack is intentionally narrow, reducing cognitive and maintenance overhead.
2. BLE behavior is anchored on a platform (nRF52840) that supports both current and future power models.
3. Python is treated as a first-class implementation language, not a prototype-only choice.
4. Hardware and runtime migration after initial bring-up is explicitly avoided.
5. Development velocity is prioritized over premature optimization.
