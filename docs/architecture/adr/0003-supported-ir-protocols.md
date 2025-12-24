# 0003: Supported IR Protocols

- Status: proposed
- Date: 2025-12-24

## Context

There are many IrDA protocols for consumer electronics, but few are well-documented. The project aims to support a wide
range of devices while maintaining a manageable implementation. There is a need for a common representation of IR
protocol payloads.

## Decision

1. **Only stateless protocols are supported.**
    * Stateful protocols (like those often used in air conditioning devices) are typically not well-documented and are
      difficult to reverse-engineer. These are out of scope.
2. **Supported Protocols:**
    * The system supports approximately 26 protocols and their varieties initially, with a capacity for up to 32
      protocols.

| PROTO_ID | Protocol Name       | Practical Notes               |
|---------:|---------------------|-------------------------------|
|        0 | **RAW**             | Raw timings / fallback        |
|        1 | **NEC**             | Classic NEC (1B addr, 1B cmd) |
|        2 | **NEC_EXT**         | Extended NEC (2B addr)        |
|        3 | **NEC2**            | NEC with full-frame repeat    |
|        4 | **RC5**             | Philips RC-5                  |
|        5 | **RC6**             | Philips RC-6                  |
|        6 | **SONY_SIRC_12**    | Sony 12-bit                   |
|        7 | **SONY_SIRC_15**    | Sony 15-bit                   |
|        8 | **SONY_SIRC_20**    | Sony 20-bit                   |
|        9 | **JVC**             | JVC (NEC-like)                |
|       10 | **SAMSUNG**         | Samsung (NEC-like)            |
|       11 | **LG**              | LG (NEC-like)                 |
|       12 | **PANASONIC**       | Panasonic                     |
|       13 | **SHARP**           | Sharp                         |
|       14 | **DENON**           | Denon                         |
|       15 | **PIONEER**         | Pioneer                       |
|       16 | **YAMAHA**          | Yamaha (NEC-like variant)     |
|       17 | **ONKYO**           | Onkyo                         |
|       18 | **MARANTZ**         | Marantz                       |
|       19 | **HARMAN_KARDON**   | Harman Kardon                 |
|       20 | **BOSE**            | Bose                          |
|       21 | **APPLE_IR**        | Apple aluminum remote         |
|       22 | **BANG_OLUFSEN**    | Bang & Olufsen                |
|       23 | **GRUNDIG**         | Grundig                       |
|       24 | **TOSHIBA**         | Toshiba                       |
|       25 | **MITSUBISHI**      | Mitsubishi                    |
|       26 | **PHILIPS_GENERIC** | Other Philips protocols       |
|   27..31 | **RESERVED**        | Future use                    |

3. **Payload Representation:**
    * The protocol payload (address, command, and rare parameters) is designed to fit within **1 to 6 (six) bytes**.
    * Most common cases (address + command) fit in 4 (four) bytes.
    * Rare cases (extended commands) fit in up to 6 (six) bytes.

## Consequences

- **Positive outcomes:**
    - Standardized way to handle most consumer IR devices.
    - Lightweight payload representation suitable for microcontroller communication.
- **Trade-offs / risks:**
    - No support for Air Conditioning units or other stateful or exotic devices.
- **Follow-ups:**
    - Implementation of specific protocol encoders/decoders in the microcontroller firmware.

## Notes

- The 6-byte payload limit is a critical design constraint for communication protocols between the frontend device (typically Android) and the IR backend.
