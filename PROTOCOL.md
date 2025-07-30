
---

## 📡 NETWORK_DISCOVERY Packet

The `NETWORK_DISCOVERY` packet is broadcast by a device attempting to join the AutoMesh network. It is transmitted over IEEE 802.15.4 using **mixed addressing**: a 64-bit extended source address and a 16-bit broadcast destination address.


### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field                 | Value                          |
|-----------------------|--------------------------------|
| Frame Type            | Data (0x0001)                  |
| Destination PAN ID    | Fixed (e.g., `0xA0A0`)         |
| Destination Address   | `0xFFFF` (broadcast)           |
| Source PAN ID         | _Omitted_ (via PAN compression)|
| Source Address        | 64-bit EUI-64                  |
| Addressing Mode       | Dst: 16-bit, Src: 64-bit       |
| PAN ID Compression    | Set to 1                       |

### 📦 AutoMesh Payload Format

The payload uses a **Type–Length–Value (TLV)** structure. For fixed-length types, the `length` field may be omitted.

| Field         | Size | Description                                      |
|---------------|------|--------------------------------------------------|
| `type`        | 1 B  | Message type (`0x01` = `NETWORK_DISCOVERY`)      |
| TLVs          | ≥1 B | One or more TLV entries (e.g., Device Role TLV)  |

### 🧠 Purpose

- Advertise device presence to nearby Routers or the Leader
- Indicate desired role (Router, End Device, etc.)
- Provide EUI-64 for identification and address assignment
- Trigger `NETWORK_RESPONSE` from eligible parent nodes

### 📡 Transmission Summary

- Sent after boot and passive scan failure
- Broadcast on current channel
- Retransmitted with exponential backoff if unanswered
- May be repeated on other channels (if supported)

---

## 📡 NETWORK_RESPONSE Packet

The `NETWORK_RESPONSE` packet is sent by Routers or the Leader in response to a `NETWORK_DISCOVERY` broadcast from a joining device. It informs the device that the responder is available as a parent, and may optionally provide metadata such as hop count or link quality.


### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field                 | Value                          |
|-----------------------|--------------------------------|
| Frame Type            | Data (0x0001)                  |
| Destination PAN ID    | Fixed (e.g., `0xA0A0`)         |
| Destination Address   | 64-bit EUI-64 (from discovery) |
| Source PAN ID         | _Omitted_ (via PAN compression)|
| Source Address        | 16-bit short address (AutoMesh)|
| Addressing Mode       | Dst: 64-bit, Src: 16-bit       |
| PAN ID Compression    | Set to 1                       |

> The responder knows the requester's EUI-64 from the `NETWORK_DISCOVERY` payload and unicasts this response directly.


### 📦 AutoMesh Payload Format

| Field         | Size | Description                                      |
|---------------|------|--------------------------------------------------|
| `type`        | 1 B  | Message type (`0x02` = `NETWORK_RESPONSE`)       |
| TLVs          | ≥1 B | One or more TLV entries                          |


### 🧠 Purpose

- Inform the requesting device that this node is eligible to be its parent
- Provide metadata (hop count, router load, link quality, etc.)
- Enable the joining device to evaluate potential parents and select the best one


### 📡 Transmission Summary

- Unicast to the EUI-64 of the requester
- May be delayed slightly to allow RSSI-based parent ranking
- May be sent by multiple Routers in parallel


## 🧩 Example TLVs in `NETWORK_RESPONSE`

### 📘 TLV Type: `0x10` – Hop Count

| Field     | Size | Description              |
|-----------|------|--------------------------|
| `type`    | 1 B  | `0x10`                   |
| `value`   | 1 B  | Hop count to Leader      |


### 📘 TLV Type: `0x11` – Router Load

| Field     | Size | Description              |
|-----------|------|--------------------------|
| `type`    | 1 B  | `0x11`                   |
| `value`   | 1 B  | 0–255 (higher = more load)|


### 📘 TLV Type: `0x12` – Signal Strength (RSSI)

| Field     | Size | Description                     |
|-----------|------|---------------------------------|
| `type`    | 1 B  | `0x12`                          |
| `value`   | 1 B  | Signed RSSI in dBm (2's comp)   |


### 🔢 Example Payload (Hex)

--- 

## 🧩 TLV Type Registry (AutoMesh v1.0)

AutoMesh protocol messages support extensible TLV encoding.

Each TLV follows this structure:

| Field     | Size          | Description                                         |
|-----------|---------------|-----------------------------------------------------|
| `type`    | 1 byte        | Globally unique field identifier                    |
| `length`  | *(optional)*  | Number of bytes in `value` (omit if fixed)         |
| `value`   | N bytes       | Field-specific binary value                         |

> **Devices must**:
> - Parse known TLV types only
> - Gracefully skip unknown types
> - Tolerate flexible field ordering

---

### 📘 TLV Type: `0x02` – Device Role

#### Purpose
Indicates the **desired functional role** of the device attempting to join.

#### TLV Format

| Field     | Size | Description                    |
|-----------|------|--------------------------------|
| `type`    | 1 B  | `0x02`                         |
| `value`   | 1 B  | Role code (see table below)    |

> _Length field omitted (fixed length = 1 byte)_

#### Role Code Values

| Value | Role                         |
|--------|------------------------------|
| `0x00` | Router (Full-function node)  |
| `0x01` | End Device (ED)              |
| `0x02` | Sleepy End Device (SED)      |

#### Example (Hex)
