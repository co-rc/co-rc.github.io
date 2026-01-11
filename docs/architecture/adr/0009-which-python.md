# ADR 0009: Choice of Python Runtime and MCU Platform

- Status: accepted
- Date: 2026-01-10

## Context

The system consists of an Android application communicating over Bluetooth Low Energy (BLE) with a single embedded
device. The initial deployment assumes a permanently powered device and prioritizes BLE stability and development
velocity.

At the same time, the architecture must support a future transition to low-power operation (sleep / wake cycles)
without requiring a migration of hardware, runtime, or the BLE protocol itself.

Additional constraints and considerations:

- BLE must be stable and predictable when used with Android.
- The solution must scale conceptually to low-power operation, even if sleep is not enabled initially.
- Mesh networking is a potential future direction, but is explicitly out of scope for the current decision.
- The project deliberately avoids ESP-based platforms.
- Hardware and runtime churn after initial bring-up is considered a high-risk outcome.

## Decision

The system will be implemented using:

- **MCU platform:** nRF52840
- **Runtime:** CircuitPython
- **Communication model:** BLE, single device, session-oriented

This choice is made with a long-term perspective: the same hardware, runtime, and BLE contract are intended to be used
both in the initial always-on configuration and in a future low-power (sleep-enabled) configuration.

The initial implementation will run continuously and will not rely on deep sleep. Sleep support may be introduced later
without changing the external BLE API or the overall software structure.

Mesh networking is explicitly out of scope for this ADR. The design may remain mesh-aware (e.g. identifiers, addressing)
but no mesh behavior is required or implemented at this stage.

### Why Python and not C++

While a C++ implementation would be entirely feasible and well within the team's expertise, Python is chosen to reduce
iteration cost and cognitive overhead while validating higher-level system behavior; given the capabilities of the
nRF52840 and the maturity of its BLE support, this is considered a reasonable and sufficient choice.

## Consequences

### Positive outcomes

- BLE behavior and APIs remain consistent from the first prototype through future low-power revisions.
- No planned migration of MCU, runtime, or BLE stack when sleep is introduced.
- Faster bring-up and iteration compared to a C++-based stack.
- CircuitPython provides a stable, high-level BLE abstraction well-suited for Android interoperability.
- The architecture naturally supports a stateless, session-based BLE interaction model.

### Trade-offs / risks

- CircuitPython restarts the VM after deep sleep; long-lived in-memory state must not be assumed.
- Less control over ultra-low-level BLE and power-management details compared to a C++ + Nordic SDK solution.
- RAM and performance overhead compared to a native implementation.

### Follow-ups

- Define a stable BLE service and characteristic contract suitable for both always-on and sleep-enabled operation.
- Establish clear rules for state persistence (what must be reconstructed on each connection).
- Optionally reserve protocol fields for future multi-node or mesh-related metadata.

## Notes

- This decision intentionally favors architectural stability over short-term optimization.
- The nRF52840 platform is selected explicitly to avoid later rework when low-power operation becomes a requirement.
- Mesh networking is a future consideration and will be addressed in a separate ADR if and when it becomes relevant.
