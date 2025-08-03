# 📡 AutoMesh Protocol – v1.0

This document defines the core packet structures and messaging formats for the **AutoMesh** protocol.

---

## 📡 DISCOVER\_REQUEST Packet

The `DISCOVER_REQUEST` packet is broadcast by a device attempting to join the AutoMesh network. It is transmitted over IEEE 802.15.4 using **mixed addressing**: a 64-bit extended source address and a 16-bit broadcast destination address.

### 🧠 Purpose

* Advertise device presence
* Indicate desired role (e.g., Router, End Device)
* Provide EUI-64 for identification
* Trigger `DISCOVER_RESPONSE` from Routers

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field               | Value                           |
| ------------------- | ------------------------------- |
| Frame Type          | Data (0x0001)                   |
| Destination PAN ID  | Fixed (e.g., `0xA0A0`)          |
| Destination Address | `0xFFFF` (broadcast)            |
| Source PAN ID       | *Omitted* (via PAN compression) |
| Source Address      | 64-bit EUI-64                   |
| Addressing Mode     | Dst: 16-bit, Src: 64-bit        |
| PAN ID Compression  | Set to 1                        |

### 📦 Payload Format

| Field      | Size | Description                                    |
| ---------- | ---- | ---------------------------------------------- |
| `type`     | 1 B  | Message type (`0x01` = discover request)       |
| TLV Fields | ≥1 B | One or more TLV entries (see [TLV.md](TLV.md)) |

**Required TLVs in `DISCOVER_REQUEST`:**

* `Device Role` (`type-len = 0x00`)
* `EUI TLV` (`type-len = 0xC0`)
* Optionally: `Router ID`, `Signal Strength (RSSI)`, or others as needed

---

## 📡 DISCOVER\_RESPONSE Packet

The `DISCOVER_RESPONSE` packet is sent by a Router or Leader in response to a `DISCOVER_REQUEST`. It informs the joining device that the responder is eligible to be a parent.

### 🧠 Purpose

* Advertise availability as a parent
* Provide link metrics for parent selection
* Allow joining device to evaluate candidates

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field               | Value                               |
| ------------------- | ----------------------------------- |
| Frame Type          | Data (0x0001)                       |
| Destination PAN ID  | Fixed (e.g., `0xA0A0`)              |
| Destination Address | 64-bit EUI-64 from discover request |
| Source PAN ID       | *Omitted*                           |
| Source Address      | 16-bit short address                |
| Addressing Mode     | Dst: 64-bit, Src: 16-bit            |
| PAN ID Compression  | Set to 1                            |

## 📦 Payload Format – `DISCOVER_RESPONSE`

| Field      | Size | Description                                    |
| ---------- | ---- | ---------------------------------------------- |
| `type`     | 1 B  | Message type (`0x02` = discover response)      |
| TLV Fields | ≥1 B | One or more TLV entries (see [TLV.md](TLV.md)) |

**Required TLVs in `DISCOVER_RESPONSE`:**

* `Device Role` (`type-len = 0x00`)
* `Router ID` (`type-len = 0x04`)
* `Partition ID` (`type-len = 0x80`)
* `Signal Strength (RSSI)` (`type-len = 0x03`)
* Optionally: `Network Weight`, `Router Load`, `Link Cost to Leader`, `Status`

---

## 📡 JOIN\_REQUEST Packet

The `JOIN_REQUEST` packet is sent by a device after selecting a Router as its preferred parent. It initiates the secure admission process to formally join the AutoMesh network.

### 🧠 Purpose

* Formally request admission to the network
* Propose parent-child relationship

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field               | Value                          |
| ------------------- | ------------------------------ |
| Frame Type          | Data (0x0001)                  |
| Destination PAN ID  | Fixed (e.g., `0xA0A0`)         |
| Destination Address | 16-bit short address of Router |
| Source PAN ID       | *Omitted*                      |
| Source Address      | 64-bit EUI-64                  |
| Addressing Mode     | Dst: 16-bit, Src: 64-bit       |
| PAN ID Compression  | Set to 1                       |

## 📦 Payload Format – `JOIN_REQUEST`

| Field      | Size | Description                                    |
| ---------- | ---- | ---------------------------------------------- |
| `type`     | 1 B  | Message type (`0x03` = join request)           |
| TLV Fields | ≥1 B | One or more TLV entries (see [TLV.md](TLV.md)) |

**Required TLVs in `JOIN_REQUEST`:**

* `Device Role` (`type-len = 0x00`)
* `EUI TLV` (`type-len = 0xC0`)
* Optionally: `Extended Address` (`type-len = 0xC1`), `Router ID`, `Status`

---

## 📡 JOIN\_RESPONSE Packet

The `JOIN_RESPONSE` packet is sent by a Router in response to a valid `JOIN_REQUEST`. It confirms admission and provides network addressing and essential network context.

### 🧠 Purpose

* Confirm join acceptance
* Assign short address
* Communicate network partition metadata

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field               | Value                           |
| ------------------- | ------------------------------- |
| Frame Type          | Data (0x0001)                   |
| Destination PAN ID  | Fixed (e.g., `0xA0A0`)          |
| Destination Address | 64-bit EUI-64 of joining device |
| Source PAN ID       | *Omitted*                       |
| Source Address      | 16-bit short address of Router  |
| Addressing Mode     | Dst: 64-bit, Src: 16-bit        |
| PAN ID Compression  | Set to 1                        |

## 📦 Payload Format – `JOIN_RESPONSE`

| Field      | Size | Description                                    |
| ---------- | ---- | ---------------------------------------------- |
| `type`     | 1 B  | Message type (`0x04` = join response)          |
| TLV Fields | ≥1 B | One or more TLV entries (see [TLV.md](TLV.md)) |

**Required TLVs in `JOIN_RESPONSE`:**

* `Assigned Short Address` (`type-len = 0x40`)
* `Partition ID` (`type-len = 0x80`)
* `Status` (`type-len = 0x07`)
* Optionally: `Router ID`, `Network Weight`, `Link Cost to Leader`

> For all packets, see [TLV.md](TLV.md) for the full registry of TLV field definitions, type-len values, and semantics.
> Devices must gracefully ignore unknown TLV types.
