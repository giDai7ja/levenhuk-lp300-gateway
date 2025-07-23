# Levenhuk Wezzer PRO LP300 Weather Station Protocol Documentation

**Frequency:** 433.925 MHz  
**Modulation:** ASK (Amplitude Shift Keying)  
**Data Format:** 20 nibbles (80 bits)  
**Checksum:** CRC8  

## Transmission Modes

The protocol supports three transmission types:

1. **Weather Data** (Primary sensor readings)
2. **DCF Time Sync** (European time standard)
3. **WWVB Time Sync** (North American time standard)

## Common Frame Structure (All Modes)

| Nibble | Name    | Bit 3         | Bit 2         | Bit 1         | Bit 0         | Description                           |
|--------|---------|---------------|---------------|---------------|---------------|---------------------------------------|
| 1      | FC      | FC_3          | FC_2          | FC_1          | FC_0          | Family Code (Device Identifier)       |
| 2      | SC_H    | SC_7          | SC_6          | SC_5          | SC_4          | Security Code (Upper nibble)          |
| 3      | SC_L    | SC_3          | SC_2          | SC_1          | SC_0          | Security Code (Lower nibble)          |
| 17     | STATUS  | dir_err       | fixed(0)      | fixed(0)      | low_bat       | System Status Flags                   |
| 18     | W_DIR   | w_dir_3       | w_dir_2       | w_dir_1       | w_dir_0       | Wind Direction (0x0-0xF)              |
| 19-20  | CRC     | CRC_7-CRC_4   | CRC_3-CRC_0   |               |               | CRC-8 Checksum                        |

## 1. Weather Data Mode

| Nibble | Name    | Bits          | Description                           | Value Interpretation          |
|--------|---------|---------------|---------------------------------------|-------------------------------|
| 4      | TMP_H   | flag1,flag0   | Temperature flags                    | `flag1:flag0` = `1:1` = error |
|        |         | TMP_9,TMP_8   | Temp high bits                       | Hex value (MSB)               |
| 5      | TMP_M   | TMP_7-TMP_4   | Temp middle bits                     | Hex value                     |
| 6      | TMP_L   | TMP_3-TMP_0   | Temp low bits                        | Hex value (LSB)               |
| 7      | HM_H    | HM_7-HM_4     | Humidity high nibble                 | Hex value                     |
| 8      | HM_L    | HM_3-HM_0     | Humidity low nibble                  | Hex value                     |
| 9      | WSP_H   | WSP_7-WSP_4   | Wind speed high nibble               | Hex value (0-255)             |
| 10     | WSP_L   | WSP_3-WSP_0   | Wind speed low nibble                | Hex value                     |
| 11     | GUST_H  | GUST_7-GUST_4 | Wind gust high nibble                | Hex value                     |
| 12     | GUST_L  | GUST_3-GUST_0 | Wind gust low nibble                 | Hex value                     |
| 13-16  | RAIN    | RAIN_15-RAIN_0| Rain counter (16 bits)               | Hex value (0-65535)           |

## 2. DCF Time Sync Mode

| Nibble | Name    | Bits          | Description                           | Values                        |
|--------|---------|---------------|---------------------------------------|-------------------------------|
| 4      | FLAG    | -             | DCF-specific flags                   | `A` for DCF mode              |
| 5      | HOUR_H  | DST_1,DST_0   | DST status                           | `01`=no DST, `10`=DST active  |
|        |         | HR_1,HR_0     | Hour tens digit (BCD)                | 0-2                           |
| 6      | HOUR_L  | HR_3-HR_0     | Hour ones digit (BCD)                | 0-9                           |
| 7-8    | MIN     | MIN_7-MIN_0   | Minutes (BCD)                        | 00-59                         |
| 9-10   | SEC     | SEC_7-SEC_0   | Seconds (BCD)                        | 00-59                         |
| 11-12  | YEAR    | YEAR_7-YEAR_0 | Year (BCD)                           | 00-99                         |
| 13     | MONTH_H | TZ_1,TZ_0     | Timezone offset bits                 |                               |
|        |         | MONTH_1,MONTH_0| Month tens digit                    | 0-1                           |
| 14     | MONTH_L | MONTH_3-MONTH_0| Month ones digit                    | 0-9                           |
| 15-16  | DAY     | DAY_7-DAY_0   | Day (BCD)                           | 01-31                         |

## 3. WWVB Time Sync Mode

| Nibble | Name    | Bits          | Description                           | Values                        |
|--------|---------|---------------|---------------------------------------|-------------------------------|
| 4      | FLAG    | -             | WWVB-specific flags                  | `5` for WWVB mode             |
| 5      | HOUR_H  | DST_1,DST_0   | DST status                           | `00`=no DST, `10`=DST start   |
|        |         | HR_1,HR_0     | Hour tens digit                      | 0-2                           |
| 6      | HOUR_L  | HR_3-HR_0     | Hour ones digit                      | 0-9                           |
| 7-16   | ...     | ...           | (Similar to DCF but WWVB-specific)    |                               |

## Wind Direction Encoding (Nibble 18)

| Hex | Direction | Degrees | Hex | Direction | Degrees |
|-----|-----------|---------|-----|-----------|---------|
| 0   | N         | 0°      | 8   | S         | 180°    |
| 1   | NNE       | 22.5°   | 9   | SSW       | 202.5°  |
| 2   | NE        | 45°     | A   | SW        | 225°    |
| 3   | NEE       | 67.5°   | B   | SWW       | 247.5°  |
| 4   | E         | 90°     | C   | W         | 270°    |
| 5   | EES       | 112.5°  | D   | WWN       | 292.5°  |
| 6   | ES        | 135°    | E   | WN        | 315°    |
| 7   | ESS       | 157.5°  | F   | WNN       | 337.5°  |

## Status Flags Details

- **dir_err (Bit 3, Nibble 17)**:  
  `1` = Wind direction sensor error, `0` = Normal operation
- **low_bat (Bit 0, Nibble 17)**:  
  `1` = Low battery, `0` = Normal voltage

## Temperature Calculation Example

For nibbles 4-6 with values `TMP_H=0x2`, `TMP_M=0x5`, `TMP_L=0xF`:

1. Combine bytes: `0x25F`
2. Convert to decimal: 607
3. Divide by 10: 60.7°C

## Checksum Verification

CRC-8 algorithm should be applied to nibbles 1-18 to verify against nibbles 19-20.

---
