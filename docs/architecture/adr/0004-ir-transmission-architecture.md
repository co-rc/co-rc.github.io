# 0004: IR Transmission Architecture

- Status: proposed
- Date: 2025-12-24

## Context

The system needs to translate user interactions on a mobile device (Android) into precise infrared signals emitted by a
microcontroller. A clear separation of concerns is required to handle high-level user intents, device-specific key
mappings, and low-level protocol encoding.

## Decision

The IR transmission process is divided into three distinct layers:

### Layer A: User Intent (Android Frontend)

- **Output:** `button_id`, `gesture_intent` (e.g., `SHORT_PRESS`, `PRESS_HOLD`, `DOUBLE_CLICK`).
- **Responsibility:** Captures user interaction and passes it to the next layer.

### Layer B: Action Mapping (Converter / Keymap)

- **Inputs:** `button_id`, `gesture_intent`.
- **Outputs:** `ir_command`, `emission_mode`.
- **Responsibility:** Maps a logical button press to a specific IR command and defines whether it should be a single
  burst or a repeated sequence.

### Layer C: IR TX (Low-level Backend)

- **Inputs:** `ir_command`, `emission_mode`.
- **Responsibility:** Executes the actual IR emission.
    - `C`: Emit a single IR frame.
    - `CCCC`: Emit a sequence of IR frames (repeats) until a timeout.
- **Note:** This layer is responsible for handling protocol-specific timing and hardware control.

## Consequences

- **Positive outcomes:**
    - Decouples UI logic from hardware-specific IR protocol details.
    - Allows for flexible key mappings (e.g., the same button can do different things based on press duration).
    - Simplifies testing by allowing each layer to be verified independently.
- **Trade-offs / risks:**
    - Slight overhead due to abstraction layers.
- **Follow-ups:**
    - Define the communication protocol between `Layer B` and `Layer C` (Microcontroller).

## Notes

- This architecture ensures that the Android application doesn't need to know the fine details of IR modulation, only
  the logical commands it wants to send.
- **Terminology:**
    - **`C` (Command):** Represents a single, complete IR frame or burst corresponding to one protocol-specific command.
    - **`CCCC` (Repeated Sequence):** Represents a sequence of the same IR frame (`C`) emitted multiple times, usually separated by a protocol-defined gap. This is typically used for "press and hold" behavior or for devices that require multiple frames to acknowledge a command.
