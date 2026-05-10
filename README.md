# Modbus Protocol – Comprehensive Guide

A complete reference and implementation guide for the Modbus communication protocol, including Modbus RTU, Modbus ASCII, and Modbus TCP/IP.

This repository is intended as a technical reference, learning resource, and implementation guide for engineers developing or integrating Modbus devices.

## Table of Contents
- [Introduction](#introduction)
- [What is Modbus](#what-is-modbus)
- [Modbus Architecture](#modbus-architecture)
- [Data Model](#data-model)
- [Addressing Model](#addressing-model)
- [Function Codes](#function-codes)
- [Modbus RTU](#modbus-rtu)
- [Modbus ASCII](#modbus-ascii)
- [Modbus TCP/IP](#modbus-tcpip)
- [Broadcast Communication](#broadcast-communication)
- [Data Limits in Modbus](#data-limits-in-modbus)
- [Register Mapping Design](#register-mapping-design)
- [Implementation Guidelines](#implementation-guidelines)
- [Interoperability Considerations](#interoperability-considerations)
- [Example Communication Frames](#example-communication-frames)
- [References](#references)
- [License](#license)
- [Contribution](#contribution)

---

## Introduction
Modbus is one of the most widely used communication protocols in industrial automation systems. It was originally developed by Modicon (now Schneider Electric) in 1979 for communication between programmable logic controllers (PLCs).

Today Modbus is used in thousands of devices including:
- PLCs
- SCADA systems
- Industrial sensors
- Motor drives
- Energy meters
- Condition monitoring systems
- Embedded controllers

One of the main reasons for its popularity is that Modbus is:
- Open
- Simple
- Easy to implement
- Widely supported

---

## What is Modbus
Modbus is a master–slave (client–server) communication protocol used to exchange data between devices.

**Typical communication flow:**
1. The **Master** sends a request.
2. The **Slave** processes the request.
3. The **Slave** returns a response.

*Note: Only the master can initiate communication.*

---

## Modbus Architecture
A Modbus network typically consists of:

**Master (Client)**
- Industrial computer, PLC, SCADA system, or controller.

**Slave (Server)**
- Devices such as sensors, controllers, analyzers, or embedded systems.

**Communication medium:**
- RS‑485
- RS‑232
- Ethernet (for Modbus TCP)

**Example topology:**
`Master → Slave1 → Slave2 → Slave3`

*Only one device acts as the master on the bus.*

---

## Data Model
Modbus organizes device data into four primary object types:

1. **Coils**
   - Single-bit values
   - Read/Write
   - Used for digital outputs
   - *Example:* LED control, Relay output

2. **Discrete Inputs**
   - Single-bit values
   - Read-only
   - Used for digital inputs
   - *Example:* Switch status, Limit sensors

3. **Holding Registers**
   - 16-bit values
   - Read/Write
   - Most commonly used data type
   - *Example:* Temperature, Configuration parameters

4. **Input Registers**
   - 16-bit values
   - Read-only
   - *Example:* Measured sensor values

---

## Addressing Model
Modbus uses 16-bit addressing, meaning each data type supports `0 – 65535` addresses.

However, documentation often uses a specific notation system:
- **Coils:** `00001 – 09999`
- **Discrete Inputs:** `10001 – 19999`
- **Input Registers:** `30001 – 39999`
- **Holding Registers:** `40001 – 49999`

**Important:**
These numbers are documentation conventions, not the actual transmitted addresses. Inside the Modbus frame, the address always starts from `0`.
- *Example:* `40001` → Address `0`
- *Example:* `40002` → Address `1`

---

## Function Codes
Function codes define the operation requested by the master.

**Common function codes:**
- `01` – Read Coils
- `02` – Read Discrete Inputs
- `03` – Read Holding Registers
- `04` – Read Input Registers
- `05` – Write Single Coil
- `06` – Write Single Register
- `15` – Write Multiple Coils
- `16` – Write Multiple Registers

*Example: Reading holding registers uses Function Code = `03`.*

---

## Modbus RTU
Modbus RTU is the most common serial implementation of Modbus.

**Characteristics:**
- Binary encoding
- Compact frames
- High efficiency
- CRC error checking

**Communication medium:** RS‑485 or RS‑232
**Typical baud rates:** 9600, 19200, 38400, 115200

### RTU Frame Structure
| Slave Address | Function Code | Data | CRC |
| :---: | :---: | :---: | :---: |
| 1 byte | 1 byte | N bytes | 2 bytes |

*Example Request Frame Structure:*
`[Slave Address] [Function Code] [Start Address] [Quantity] [CRC]`

---

## Modbus ASCII
Modbus ASCII is an alternative serial transmission mode.

**Characteristics:**
- ASCII character encoding
- Human-readable messages
- Lower communication efficiency

### ASCII Frame Structure
| Start Character | Address | Function | Data | LRC Checksum | End Characters |
| :---: | :---: | :---: | :---: | :---: | :---: |
| `:` | 2 chars | 2 chars | N chars | 2 chars | `CR LF` |

*Example:* `:010300000002FA`

**Advantages:** Easy debugging, Readable messages
**Disadvantages:** Lower throughput compared to RTU

---

## Modbus TCP/IP
Modbus TCP/IP runs on Ethernet networks using TCP.

**Communication port:** TCP Port `502`

*Unlike serial Modbus, Modbus TCP does not use CRC because the underlying TCP layer already provides error checking.*

### TCP Frame Structure
`[MBAP Header] + [PDU]`

**MBAP Header contains:**
- Transaction Identifier
- Protocol Identifier
- Length
- Unit Identifier

**PDU contains:**
- Function Code
- Data

**Advantages:** High speed, Ethernet integration, Remote communication.

---

## Broadcast Communication
Modbus supports broadcast messaging.

**Broadcast address:** `0`

When the master sends a message to address `0`, all slaves receive it.
- All slaves execute the command.
- No slave sends a response.

**Broadcast is typically used for:**
- Writing configuration values
- Synchronizing commands
- Reset commands

*Supported in:* Modbus RTU, Modbus ASCII.
*Not generally used in:* Modbus TCP networks.

---

## Data Limits in Modbus
Although the address space supports up to 65536 registers, the protocol limits how much data can be transferred in a single message.

For **Modbus RTU**, the maximum frame size is ≈ 256 bytes.

**Typical limits:**
- **Read Holding Registers (FC03):** Maximum 125 registers
- **Read Input Registers (FC04):** Maximum 125 registers
- **Write Multiple Registers (FC16):** Maximum 123 registers

*If more data is required, the master must send multiple sequential requests.*

---

## Register Mapping Design
When designing a Modbus device, a register map must be clearly defined.

### Example Design: Holding Registers
| Range | Description |
|---|---|
| `0 – 9` | Sensor values |
| `10 – 19` | Calculated values |
| `100 – 119` | Configuration parameters |
| `200 – 209` | Alarm thresholds |

### Example Mapping Table
| Address | Data Type | Description |
|:---:|:---:|---|
| `0` | Holding Register | Temperature |
| `1` | Holding Register | Pressure |
| `2` | Holding Register | Speed |
| `100` | Holding Register | Device Mode |
| `101` | Holding Register | Sampling Rate |
| `0` | Coil | LED1 |
| `1` | Coil | Relay1 |
| `2` | Coil | Reset Alarm |

---

## Implementation Guidelines
When implementing Modbus in an embedded system:
- Define a clear register map.
- Separate measurement data from configuration parameters.
- Ensure correct endian format (Modbus is typically Big-Endian).
- Implement CRC correctly.
- Support common function codes.
- Handle invalid requests gracefully (return Exception Codes).
- Implement timeouts and retries.

---

## Interoperability Considerations
To ensure compatibility with different masters:
- Follow standard function codes.
- Avoid vendor-specific extensions unless strictly documented.
- Provide a clear and publicly available register map.
- Support standard baud rates and serial settings (e.g., 8N1, 8E1).
- Document data types and scaling factors clearly.

---

## Example Communication Frames
**Scenario:** Reading temperature from register `0`.

**Master Request (RTU):**
```text
Slave Address: 01
Function: 03
Start Address: 0000
Quantity: 0001
CRC: [Calculated Bytes]

**Slave Response (RTU):**
text
Slave Address: 01
Function: 03
Byte Count: 02
Register Value: [Data Bytes]
CRC: [Calculated Bytes]

```
---

## References
- **Modbus Organization:** [https://modbus.org](https://modbus.org)
- Modbus Application Protocol Specification
- Modbus over Serial Line Specification
- Industrial communication references and device documentation.

---

## License
[MIT License](LICENSE)

## Contribution
Contributions are welcome! You can contribute by:
- Adding implementation examples
- Improving documentation
- Providing specific device register maps
- Implementing or referencing Modbus libraries
