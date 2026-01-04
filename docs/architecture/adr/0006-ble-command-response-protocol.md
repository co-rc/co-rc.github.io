# ADR 0006: BLE Command/Response Protocol Specification

- Status: implemented
- Date: 2026-01-04

## Context

Android BLE (GATT) is callback-driven and limited when performing multiple concurrent operations. To ensure reliability
and simplify the application logic, we need:

- Deterministic serialization of all GATT operations.
- A modern asynchronous API for the Android client.
- A robust framing protocol for application-level commands and responses.

## Decision

We implement a serialized, future-based command/response system with the following characteristics:

### Infrastructure

- **Android Central**: Java 21, `minSdk = 36`.
- **No Reactive Extensions**: No `Flow.Publisher`, no RxJava; using native `CompletableFuture`.
- **Serialization**: `BleOperationQueue` ensures only one GATT operation is active at a time.
- **Async API**: All operations return `CompletableFuture<T>`.

### Transport Layer

- **Requests**: Issued via **Write With Response** on a dedicated `CMD` characteristic.
- **Responses**: Delivered via **GATT Notifications (NOTIFY)** on a dedicated `RSP` characteristic.

### Binary Framing Protocol

All frames are Little-Endian.

#### Request Frame (Android → Device)

| Field     | Size (Bytes) | Notes                          |
|-----------|--------------|--------------------------------|
| Magic     | 2            | `0xC07C`                       |
| RequestId | 1            | Incremented modulo 256         |
| Opcode    | 1            | Command identifier             |
| Len       | 1            | Payload length (N)             |
| Payload   | N            | Max 15 bytes (for default MTU) |

#### Response Frame (Device → Android)

| Field     | Size (Bytes) | Notes                          |
|-----------|--------------|--------------------------------|
| Magic     | 2            | `0xC07C`                       |
| RequestId | 1            | Matches RequestId              |
| Opcode    | 1            | Matches Opcode                 |
| Result    | 1            | Result code (0x00 = OK)        |
| Len       | 1            | Payload length (N)             |
| Payload   | N            | Max 14 bytes (for default MTU) |

### Result Codes

- `0x00 OK`: Success.
- `0x01 UNSUPPORTED`: Unknown opcode.
- `0x02 BAD_PARAM`: Invalid payload format or data.
- `0x03 INVALID_STATE`: Command not allowed in current device state.
- `0x04 BUSY`: Device is processing another request.
- `0x05 NOT_AUTHORIZED`: Authentication/Authorization failed.
- `0x06 INTERNAL_ERROR`: Unexpected hardware or software failure.

## Consequences

- **Reliability**: Eliminates "GATT busy" errors and race conditions.
- **Simplicity**: Application code uses flat asynchronous chains instead of nested callbacks.
- **MTU Dependency**: Without MTU negotiation, frames are limited to 20 bytes (GATT default 23 - 3 bytes overhead).
- **Correlation**: `RequestId` and `Opcode` allow precise matching of responses to requests.

## Notes

- Timeouts are managed by the `OperationQueue`. If a response notification is not received after a successful write, the
  operation eventually times out, causing a GATT disconnect to reset the stack.
- Frames exceeding the MTU limit must be chunked at the application level.
