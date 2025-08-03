# AutoMesh Protocol: Type–Length–Value (TLV) Field Reference

This page documents all currently defined TLV (Type–Length–Value) fields used in the AutoMesh protocol.

## TLV Format

| Field       | Size    | Description                                                               |
|-------------|---------|---------------------------------------------------------------------------|
| type-len    | 1 byte  | Field identifier and length specifier (see below)                         |
| value       | N bytes | Value, **variable-length as specified by the highest 2 bits of type-len** |

### `type-len` Byte Structure

The `type-len` byte is split into two parts:

- **[7:6] Length field (2 bits):** Specifies the length of the value field:
    - `00`: 1 byte  
    - `01`: 2 bytes  
    - `10`: 4 bytes  
    - `11`: 8 bytes  
- **[5:0] Type field (6 bits):** Identifies the TLV type, starting from `0x00` and increasing _separately for each length group_ (see tables below).

```
  7   6   5   4   3   2   1   0
+---+---+-----------------------+
|LEN|LEN|      TYPE[5:0]        |
+---+---+-----------------------+
```

> **General Rules:**
> - Devices must parse known types and gracefully ignore unknown types.
> - TLVs may appear in any order within a payload.
> - All values are in network byte order (big-endian).
> - **The length of the value is determined dynamically by the highest two bits of the type-len byte in each instance.**
> - **Type IDs for each length group start at `0x00` and are assigned independently.**

---

## 1-Byte TLVs (`type-len[7:6]` = `00`)

| Type ID (`type-len[5:0]`) | type-len | Name                    | Description                                                        |
|:-------------------------:|:--------:|-------------------------|--------------------------------------------------------------------|
| `0x00`                    | `0x00`   | Device Role             | `0x00`=Router, `0x01`=ED (End Device), `0x02`=SED (Sleepy ED)      |
| `0x01`                    | `0x01`   | Hop Count               | Hop count to Leader                                                |
| `0x02`                    | `0x02`   | Router Load             | 0–255 (higher = more load)                                         |
| `0x03`                    | `0x03`   | Signal Strength (RSSI)  | Signed RSSI in dBm (2's complement)                                |
| `0x04`                    | `0x04`   | Router ID               | 8-bit unique Router ID                                             |
| `0x05`                    | `0x05`   | Network Weight          | Optional quality metric (e.g., path cost, load + RSSI)             |
| `0x06`                    | `0x06`   | Link Cost to Leader     | Estimated local cost to Leader from the device's perspective       |
| `0x07`                    | `0x07`   | Status                  | `0x00` = OK, `0x01` = Rejected                                     |

---

## 2-Byte TLVs (`type-len[7:6]` = `01`)

| Type ID (`type-len[5:0]`) | type-len | Name                    | Description                                |
|:-------------------------:|:--------:|-------------------------|--------------------------------------------|
| `0x00`                    | `0x40`   | Assigned Short Address  | Assigned short (16-bit) address            |

---

## 4-Byte TLVs (`type-len[7:6]` = `10`)

| Type ID (`type-len[5:0]`) | type-len | Name         | Description                                |
|:-------------------------:|:--------:|--------------|--------------------------------------------|
| `0x00`                    | `0x80`   | Partition ID | 32-bit mesh partition ID                   |

---

## 8-Byte TLVs (`type-len[7:6]` = `11`)

| Type ID (`type-len[5:0]`) | type-len | Name             | Description                                                      |
|:-------------------------:|:--------:|------------------|------------------------------------------------------------------|
| `0x00`                    | `0xC0`   | EUI TLV          | 64-bit Extended Unique Identifier (EUI-64) assigned to the device|
| `0x01`                    | `0xC1`   | Extended Address | 8-byte IEEE 802.15.4 Extended Address (MAC address)              |

---

> **How type-len is calculated:**  
> - The actual TLV type-len byte is: `type-len = (length << 6) | type_id`
> - For example, Partition ID is 4 bytes: length bits = `10` (`0x80`), type_id = `0x00` → type-len = `0x80`
> - Assigned Short Address is 2 bytes: length bits = `01` (`0x40`), type_id = `0x00` → type-len = `0x40`
> - Device Role is 1 byte: length bits = `00` (`0x00`), type_id = `0x00` → type-len = `0x00`

> **Note:**  
> Type IDs are unique _within_ each length group.  
> The same numerical Type ID may appear in different length groups, but their meaning and usage are completely separate.

---

## Extensibility

- Devices must ignore unknown TLV types.
- TLV fields are extensible; new types may be added in future revisions.
- For further details, see the [PROTOCOL.md](PROTOCOL.md) document.
