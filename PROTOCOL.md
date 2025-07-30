# 📡 AutoMesh Protocol – v1.0

This document defines the core packet structures and messaging formats for the **AutoMesh** protocol.

---

## 📡 NETWORK_DISCOVERY Packet

The `NETWORK_DISCOVERY` packet is broadcast by a device attempting to join the AutoMesh network. It is transmitted over IEEE 802.15.4 using **mixed addressing**: a 64-bit extended source address and a 16-bit broadcast destination address.

### 🧠 Purpose

- Advertise device presence
- Indicate desired role (e.g., Router, End Device)
- Provide EUI-64 for identification
- Trigger `NETWORK_RESPONSE` from Routers

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

### 📦 Payload Format

| Field         | Size | Description                          |
|---------------|------|--------------------------------------|
| `type`        | 1 B  | Message type (`0x01` = discovery)    |
| TV Fields     | ≥1 B | One or more Type–Value entries       

---

## 📡 NETWORK_RESPONSE Packet

The `NETWORK_RESPONSE` packet is sent by a Router or Leader in response to a `NETWORK_DISCOVERY`. It informs the joining device that the responder is eligible to be a parent.

### 🧠 Purpose

- Advertise availability as a parent  
- Provide link metrics for parent selection  
- Allow joining device to evaluate candidates  

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field                 | Value                           |
|-----------------------|---------------------------------|
| Frame Type            | Data (0x0001)                   |
| Destination PAN ID    | Fixed (e.g., `0xA0A0`)          |
| Destination Address   | 64-bit EUI-64 from discovery    |
| Source PAN ID         | _Omitted_                       |
| Source Address        | 16-bit short address            |
| Addressing Mode       | Dst: 64-bit, Src: 16-bit        |
| PAN ID Compression    | Set to 1                        |

## 📦 Payload Format – `NETWORK_RESPONSE`

| Field         | Size | Description                              |
|---------------|------|------------------------------------------|
| `type`        | 1 B  | Message type (`0x02` = response)         |
| TV Fields     | ≥1 B | One or more Type–Value entries           |

### 📘 Supported TV Fields

| Type   | Name                  | Size | Description                                                  |
|--------|-----------------------|------|--------------------------------------------------------------|
| `0x14` | Network Weight        | 1 B  | Optional quality metric (e.g., path cost, load + RSSI)       |
| `0x15` | Partition ID          | 2 B  | 16-bit mesh partition ID                                     |
| `0x16` | Link Cost to Leader   | 1 B  | Estimated local cost to Leader from the new device's view    |

> All values are in network byte order (big-endian).  
> Devices must gracefully ignore unknown field types.

---

## 📡 JOIN_REQUEST Packet

The `JOIN_REQUEST` packet is sent by a device after selecting a Router as its preferred parent. It initiates the secure admission process to formally join the AutoMesh network.

### 🧠 Purpose

- Formally request admission to the network  
- Propose parent-child relationship  

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field                 | Value                             |
|-----------------------|-----------------------------------|
| Frame Type            | Data (0x0001)                     |
| Destination PAN ID    | Fixed (e.g., `0xA0A0`)            |
| Destination Address   | 16-bit short address of Router    |
| Source PAN ID         | _Omitted_                         |
| Source Address        | 64-bit EUI-64                     |
| Addressing Mode       | Dst: 16-bit, Src: 64-bit          |
| PAN ID Compression    | Set to 1                          |

## 📦 Payload Format – `JOIN_REQUEST`

| Field         | Size | Description                              |
|---------------|------|------------------------------------------|
| `type`        | 1 B  | Message type (`0x03` = join request)     |
| TV Fields     | ≥1 B | One or more Type–Value entries           |

### 📘 Supported TV Fields

| Type   | Name         | Size | Description                          |
|--------|--------------|------|--------------------------------------|
| `0x02` | Device Role  | 1 B  | `0x00`=Router, `0x01`=ED, `0x02`=SED |

---

## 📡 JOIN_RESPONSE Packet

The `JOIN_RESPONSE` packet is sent by a Router in response to a valid `JOIN_REQUEST`. It confirms admission and provides network addressing and essential network context.

### 🧠 Purpose

- Confirm join acceptance  
- Assign short address  
- Communicate network partition metadata  

### 🔧 MAC Layer Configuration (IEEE 802.15.4)

| Field                 | Value                             |
|-----------------------|-----------------------------------|
| Frame Type            | Data (0x0001)                     |
| Destination PAN ID    | Fixed (e.g., `0xA0A0`)            |
| Destination Address   | 64-bit EUI-64 of joining device   |
| Source PAN ID         | _Omitted_                         |
| Source Address        | 16-bit short address of Router    |
| Addressing Mode       | Dst: 64-bit, Src: 16-bit          |
| PAN ID Compression    | Set to 1                          |

## 📦 Payload Format – `JOIN_RESPONSE`

| Field         | Size | Description                              |
|---------------|------|------------------------------------------|
| `type`        | 1 B  | Message type (`0x04` = join response)    |
| TV Fields     | ≥1 B | One or more Type–Value entries           |

### 📘 Supported TV Fields

| Type   | Name                | Size | Description                                              |
|--------|---------------------|------|----------------------------------------------------------|
| `0x32` | Status              | 1 B  | `0x00` = OK, `0x01` = Rejected                           |
| `0x30` | Assigned Short Addr | 2 B  | Assigned address (only present if Status ≠ `0x01`)       |
| `0x15` | Partition ID        | 2 B  | 16-bit ID of the current partition (if Status ≠ `0x01`)  |

> Devices must treat a missing `Status` field as an implicit accept (`0x00`).  
> If `Status == 0x01` (Rejected), both `Assigned Short Addr` and `Partition ID` must be omitted.

---

## 🧩 Type–Value (TV) Field Registry

AutoMesh messages support extensible Type–Value encoding. Each field includes:

| Field     | Size  | Description                          |
|-----------|-------|--------------------------------------|
| `type`    | 1 B   | Field identifier                     |
| `value`   | N B   | Value (fixed-length or self-described)|

> **Rules**:
> - Devices parse known types
> - Ignore unknown types gracefully
> - Allow out-of-order TV entries

### 📘 TV Type: `0x02` – Device Role

| Type | Value | Description                     |
|------|-------|---------------------------------|
| 0x02 | 1 B   | `0x00`=Router, `0x01`=ED, `0x02`=SED |

### 📘 TV Type: `0x10` – Hop Count

| Type | Value | Description                     |
|------|-------|---------------------------------|
| 0x10 | 1 B   | Hop count to Leader             |

### 📘 TV Type: `0x11` – Router Load

| Type | Value | Description                     |
|------|-------|---------------------------------|
| 0x11 | 1 B   | 0–255 (higher = more load)      |

### 📘 TV Type: `0x12` – Signal Strength (RSSI)

| Type | Value | Description                     |
|------|-------|---------------------------------|
| 0x12 | 1 B   | Signed RSSI in dBm (2's comp)   |

### 📘 TV Type: `0x13` – Router ID

| Type | Value | Description                     |
|------|-------|---------------------------------|
| 0x13 | 1 B   | 8-bit unique Router ID          |

---

## 🧪 Example NETWORK_RESPONSE Payload (Hex)

