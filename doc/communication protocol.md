# Levenhuk Wezzer PRO LP300 Weather Station Protocol Specification

## Technical Parameters

- **Carrier Frequency:** 433.925 MHz
- **Modulation:** ASK (Amplitude Shift Keying)
- **Packet Length:** 20 nibbles (80 bits)
- **Checksum Algorithm:** CRC-8

## Protocol Architecture

### 1. Family Code Identification (Nibble 1)

All transmissions begin with a 4-bit Family Code that determines the payload structure:

| FC_3 | FC_2 | FC_1 | FC_0 | Hex | Mode Type          | Payload Description              |
|------|------|------|------|-----|--------------------|----------------------------------|
| 1    | 0    | 1    | 0    | 0xA | Weather Data       | Sensor measurements              |
| 1    | 0    | 1    | 1    | 0xB | Time Synchronization | Clock data (DCF/WWVB)           |

### 2. Security Code (Nibbles 2-3)

8-bit unique device identifier:

| Nibble | Bit 3 | Bit 2 | Bit 1 | Bit 0 | Description          |
|--------|-------|-------|-------|-------|----------------------|
| 2 (SC_H) | SC_7  | SC_6  | SC_5  | SC_4  | Upper address nibble |
| 3 (SC_L) | SC_3  | SC_2  | SC_1  | SC_0  | Lower address nibble |

## Weather Data Mode (FC=0xA)

### Complete Bit Allocation

| Nibble | Field    | Bit 3    | Bit 2    | Bit 1    | Bit 0    | Data Type | Description                     |
|--------|----------|----------|----------|----------|----------|-----------|---------------------------------|
| 4      | TMP_H    | flag1    | flag0    | TMP_9    | TMP_8    | Flags+MSB | Temperature status & high bits  |
| 5      | TMP_M    | TMP_7    | TMP_6    | TMP_5    | TMP_4    | uint4     | Temperature middle bits         |
| 6      | TMP_L    | TMP_3    | TMP_2    | TMP_1    | TMP_0    | uint4     | Temperature low bits            |
| 7      | HM_H     | HM_7     | HM_6     | HM_5     | HM_4     | uint4     | Humidity high nibble            |
| 8      | HM_L     | HM_3     | HM_2     | HM_1     | HM_0     | uint4     | Humidity low nibble             |
| 9      | WSP_H    | WSP_7    | WSP_6    | WSP_5    | WSP_4    | uint4     | Wind speed high nibble          |
| 10     | WSP_L    | WSP_3    | WSP_2    | WSP_1    | WSP_0    | uint4     | Wind speed low nibble           |
| 11     | GUST_H   | GUST_7   | GUST_6   | GUST_5   | GUST_4   | uint4     | Wind gust high nibble           |
| 12     | GUST_L   | GUST_3   | GUST_2   | GUST_1   | GUST_0   | uint4     | Wind gust low nibble            |
| 13     | RAIN_HH  | RAIN_15  | RAIN_14  | RAIN_13  | RAIN_12  | uint4     | Rain counter bits 15-12         |
| 14     | RAIN_H   | RAIN_11  | RAIN_10  | RAIN_9   | RAIN_8   | uint4     | Rain counter bits 11-8          |
| 15     | RAIN_M   | RAIN_7   | RAIN_6   | RAIN_5   | RAIN_4   | uint4     | Rain counter bits 7-4           |
| 16     | RAIN_L   | RAIN_3   | RAIN_2   | RAIN_1   | RAIN_0   | uint4     | Rain counter bits 3-0           |
| 17     | STATUS   | dir_err  | 0        | 0        | low_bat  | Flags     | System status flags             |
| 18     | W_DIR    | w_dir_3  | w_dir_2  | w_dir_1  | w_dir_0  | enum4     | Wind direction (0x0-0xF)        |
| 19     | CRC_H    | CRC_7    | CRC_6    | CRC_5    | CRC_4    | uint4     | Checksum high nibble            |
| 20     | CRC_L    | CRC_3    | CRC_2    | CRC_1    | CRC_0    | uint4     | Checksum low nibble             |

### Temperature Interpretation

1. Combine bits: `TMP_9:TMP_0` (10-bit value)
2. Convert to decimal
3. Apply formula: `Temperature = (raw_value / 10) - 40` °C

Example: `0x27A` → 634 → (634/10)-40 = 23.4°C

### Humidity Calculation

`Humidity = (HM_H << 4 | HM_L)` %

## Time Synchronization Modes (FC=0xB)

### DCF/WWVB Differentiation

| Mode | Nibble 4 Value | Identification                  |
|------|----------------|----------------------------------|
| DCF  | 0xA (1010)     | European time standard          |
| WWVB | 0x5 (0101)     | North American time standard    |

### DCF Time Format Details

| Nibble | Field    | Bits          | Description                     | Values          |
|--------|----------|---------------|---------------------------------|-----------------|
| 5      | HOUR_H   | DST_1,DST_0   | Daylight Saving Time status     | 01=Off, 10=On   |
|        |          | HR_1,HR_0     | Hour (tens place)               | 0-2             |
| 6      | HOUR_L   | HR_3-HR_0     | Hour (ones place)               | 0-9             |
| 7      | MIN_H    | MIN_7-MIN_4   | Minute (tens place)             | 0-5             |
| 8      | MIN_L    | MIN_3-MIN_0   | Minute (ones place)             | 0-9             |
| 9      | SEC_H    | SEC_7-SEC_4   | Second (tens place)             | 0-5             |
| 10     | SEC_L    | SEC_3-SEC_0   | Second (ones place)             | 0-9             |
| 11     | YEAR_H   | YR_7-YR_4     | Year (tens place)               | 0-9             |
| 12     | YEAR_L   | YR_3-YR_0     | Year (ones place)               | 0-9             |
| 13     | MONTH_H  | TZ_1,TZ_0     | Timezone offset                 | See table below |
|        |          | MON_1,MON_0   | Month (tens place)              | 0-1             |
| 14     | MONTH_L  | MON_3-MON_0   | Month (ones place)              | 0-9             |
| 15     | DAY_H    | DAY_7-DAY_4   | Day (tens place)                | 0-3             |
| 16     | DAY_L    | DAY_3-DAY_0   | Day (ones place)                | 0-9             |

### Timezone Encoding (DCF)

| TZ_1 | TZ_0 | Offset | Timezone          |
|------|------|--------|-------------------|
| 0    | 0    | +0     | UTC               |
| 0    | 1    | +1     | CET               |
| 1    | 0    | +2     | EET               |
| 1    | 1    | -1     | Reserved          |

## System Flags

### Status Byte (Nibble 17)

| Bit | Name    | Value | Description                  |
|-----|---------|-------|------------------------------|
| 3   | dir_err | 1     | Wind direction sensor fault  |
| 2   | -       | 0     | Reserved (always 0)          |
| 1   | -       | 0     | Reserved (always 0)          |
| 0   | low_bat | 1     | Battery voltage < 2.4V       |

## Wind Direction Table

| Hex | Compass | Degrees | Hex | Compass | Degrees |
|-----|---------|---------|-----|---------|---------|
| 0   | N       | 0°      | 8   | S       | 180°    |
| 1   | NNE     | 22.5°   | 9   | SSW     | 202.5°  |
| 2   | NE      | 45°     | A   | SW      | 225°    |
| 3   | NEE     | 67.5°   | B   | SWW     | 247.5°  |
| 4   | E       | 90°     | C   | W       | 270°    |
| 5   | EES     | 112.5°  | D   | WWN     | 292.5°  |
| 6   | ES      | 135°    | E   | WN      | 315°    |
| 7   | ESS     | 157.5°  | F   | WNN     | 337.5°  |

## Checksum Calculation

CRC-8 algorithm applied to nibbles 1-18:

1. Initialize CRC = 0x00
2. For each nibble:
   - XOR with CRC
   - Process through polynomial 0x07 (x^8 + x^2 + x + 1)
3. Final CRC must match nibbles 19-20

## Example Packet Decoding

**Weather Data Packet:**
`A1 34 5F 02 17 8A 42 91 00 3C 00 2A 00 00 00 00 00 0D 4E 3B`

1. FC=0xA → Weather Mode
2. Temperature: 
   - Combine 0x02, 0x17, 0x8A → 0x2178A (mask to 12-bit: 0x178A)
   - Value: (6026/10)-40 = 562.6°C (invalid, flag1:flag0=1:1 indicates sensor error)
3. Humidity: 0x42 → 66%
4. Wind Speed: 0x91 → 145 units
5. Status: dir_err=0, low_bat=0 (normal operation)
6. Wind Direction: 0x0D → WWN (292.5°)
7. CRC: Valid (calculated 0x4E3B matches packet)
