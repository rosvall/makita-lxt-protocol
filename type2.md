# Battery type 2

## Identification
Command: `cc dc 0a`

If the battery does not support `cc dc 0b`, but after being put into test mode does support `cc dc 0a`, it is type 2.

The command is supported, if the last byte of the response is 0x06.

### Response (17 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |        15 | ?           |
|        16  |        16 | Always `06` |

Note: For batteries that support both `cc dc 0a` and `cc dc 0b`, the response is the identical.

## Enter test mode *(same for type 0, 2 and 3)*
Command: `cc d9 96 a5`

### Response (1 byte)
Always `06`

## Exit test mode *(same for type 0, 2 and 3)*
Command: `cc d9 ff ff`

### Response (1 byte)
Always `06`

## Read model string (unknown if supported)
Command: `cc dc 0c`

### Response (16 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |        15 | Battery model as null-terminated ASCII string |

## Overdischarge count
Command: `cc d6 8d 05 01`

### Response (2 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         0 | Integer count of overdischarge events |
|         1  |         1 | Always `06` |


## Health
Command: `cc d6 04 05 02`

### Response (3 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | "Health" as 16 bit little endian integer |
|         2  |         2 | Always `06` |
BTC04 divides this value by battery capacity.


## Overload counters
Command: `cc d6 5f 05 07`

### Response (8 bytes)
| Byte | Description     |
| ---- | --------------- |
|    0 | Counter A       |
|    1 | ?               |
|    2 | Counter B       |
|    3 | Counter C       |
|    4 | ?               |
|    5 | Counter D       |
|    6 | Counter E       |
|    7 | Always `06`     |

In the BTC04, the 5 counters are added together. They might count different types of overload events.
Counter D is stored separately though.


## Temperature *(same for type 0, 2 and 3)*
Command: `cc d7 0e 00 02`

### Response (3 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Temperature in 1/10 K, as little endian integer |
|         2  |         2 | Always `06` |


