# 0005: IR Command Payload Specification

- Status: proposed
- Date: 2025-12-24

## Context

A standardized data structure is needed to describe IR commands sent from the frontend to the backend. This structure
must handle protocol identification, the actual payload, and any transmission constraints or timing overrides. Some
devices (e.g., Panasonic TVs) require specific emission patterns (like mandatory repeats) to recognize a command.

## Decision

We define a `Payload` record that encapsulates all information required to emit one IR command.

### `Payload` record

| Field name         | Type                       | Required | Default   | Comment                                                                                                        |
|--------------------|----------------------------|:--------:|-----------|----------------------------------------------------------------------------------------------------------------|
| `protoId`          | `PROTO_ID` (enum)          |   yes    | —         | Identifies the IR protocol family (e.g., NEC, RC5, Panasonic). See [ADR 0003](0003-supported-ir-protocols.md). |
| `flags`            | `TxFlags` (bitset)         |   yes    | `NONE`    | Transmission-related toggles. Currently supports `HAS_GAP_OVERRIDE` and `HAS_DURATION_OVERRIDE`.               |
| `requiredEmission` | `RequiredEmission` (enum)  |    no    | `DEFAULT` | Hard constraint on emission pattern (see below). This is command-level, not gesture-level.                     |
| `timing`           | `TimingOverride?`          |    no    | `null`    | Optional timing overrides, present if any timing flag is set.                                                  |
| `protoPayload`     | `ByteString` (length 1..6) |   yes    | —         | Protocol-specific data (address, command, etc.). Max 6 bytes.                                                  |

### `RequiredEmission` enum

Used to enforce a specific emission behavior regardless of the higher-layer mapping.

| Value            | Description                                                                    |
|:-----------------|--------------------------------------------------------------------------------|
| `DEFAULT`        | No specific constraint; follow the requested pattern from the higher layer.    |
| `REQUIRE_SINGLE` | Always emit exactly one frame (`C`), even if a repeat is requested.            |
| `REQUIRE_REPEAT` | Always emit a repeated sequence (`CCCC`), even if a single burst is requested. |

### `TimingOverride` structure

| Field name | Type        | Required | Default | Comment                                                                                                     |
|------------|-------------|:--------:|---------|-------------------------------------------------------------------------------------------------------------|
| `gap`      | `Duration?` |    no    | `null`  | Inter-frame gap used between consecutive frames.                                                            |
| `duration` | `Duration?` |    no    | `null`  | Total time budget for `CCCC` sequences. Only used if `HAS_DURATION_OVERRIDE` or `REQUIRE_REPEAT` is active. |

## Consequences

- **Positive outcomes:**
    - Provides a robust way to handle "sticky" commands (like Panasonic PowerOn) that need special treatment.
    - Allows fine-tuning of IR transmission for problematic devices without changing the core protocol implementation.
- **Trade-offs / risks:**
    - Increased complexity in the payload structure.
- **Follow-ups:**
    - Implementation of this record in both Android and Microcontroller environments.

## Notes

- `Duration` is represented in a language-specific standard format (e.g., milliseconds or microseconds).
- If `timing` is `null`, the backend uses protocol-defined default values.
- **Terminology:**
    - **`C` (Command):** Represents a single, complete IR frame or burst corresponding to one protocol-specific command.
    - **`CCCC` (Repeated Sequence):** Represents a sequence of the same IR frame (`C`) emitted multiple times, usually separated by a protocol-defined gap.
