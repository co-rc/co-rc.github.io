# 0002: Platforms and Languages

- Status: accepted
- Date: 2025-12-24

## Context

Fundamental decisions regarding the technology stack.

## Decision

### Team

- Currently, a single-developer project.
- Contributions are welcome — please reach out.

### Platforms

#### Microcontrollers

- Raspberry Pi Pico: Primary choice.
- NRF52840: Preferred for low power consumption and BLE capabilities.
- ESP32: Secondary option.
- Raspberry Pi (Full OS): Not funny, not considered for this scope.

#### Mobile

- Android: Target minSdk 36. No backward compatibility, sorry.
- iPhone: Not planned (no hardware available for testing).

#### Desktop

- Windows: Possible future consideration.
- Linux: Not planned.

### Languages

#### Microcontrollers

- Python:
    - MicroPython: Primary choice.
    - CircuitPython: May be better for NRF52840.
- C++: Fallback if Python is not enough (e.g., performance, power consumption, BT stack limitations).

#### Mobile

- Kotlin: Not planned.
- Java (Android): Java 21.

### IDEs

- Android/Java: IntelliJ Ultimate + Android plugin.
- Microcontrollers/python: PyCharm + MicroPython plugin.

## Consequences

1. Limited capacity: Progress depends on a single person's availability.
2. Simplified GitHub Flow: Direct commits to main, no pull requests or reviews.
3. Fast decision-making: Minimal overhead for architectural changes.
4. Shared responsibility: One person handles development, testing, and operations.
