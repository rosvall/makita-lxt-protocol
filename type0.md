# Battery type 0

## Identification
Command: `cc dc 0b`

If battery supports this command, it is type 0.

The command is supported, if the last byte of the response is `06`.

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

## Read model string
Command: `cc dc 0c`

### Response (16 bytes)
| First byte | Last byte | Description                                   |
| ---------- | --------- | --------------------------------------------- |
|         0  |        15 | Battery model as null-terminated ASCII string |


## Overdischarge count
Command: `cc d4 ba 00 01`

### Response (2 bytes)
| First byte | Last byte | Description                           |
| ---------- | --------- | ------------------------------------- |
|         0  |         0 | Integer count of overdischarge events |
|         1  |         1 | Always `06`                           |


## Health
Command: `cc d4 50 01 02`

### Response (3 bytes)
| First byte | Last byte | Description                            |
| ---------- | --------- | -------------------------------------- |
|         0  |         1 | Health as 16 bit little endian integer |
|         2  |         2 | Always `06`                            |
BTC04 divides this value by battery capacity.

## Overload counters
Command: `cc d4 8d 00 07`

### Response (8 bytes)
| Byte | Bits | Description                      |
| ---- | ---- | -------------------------------- |
|    0 | 0..5 | ?                                |
|    0 | 6..7 | Counter A (10 bit), low 2 bits   |
|    1 | 0..7 | Counter A (10 bit), high 8 bits  |
|    2 | 0..7 | ?                                |
|    3 | 0..7 | Counter B (10 bit), low 8 bits   |
|    4 | 0..1 | Counter B (10 bit), high 2 bits  |
|    4 | 2..7 | ?                                |
|    5 | 0..3 | ?                                |
|    5 | 4..7 | Counter C (10 bit), low 4 bits   |
|    6 | 0..5 | Counter C (10 bit), high 6 bits  |
|    6 | 6..7 | ?                                |
|    7 | 0..7 | Always `06`                      |

In the BTC04, the three counters are added together. They might count different types of overload events.
Counter C is stored separately though.



## Temperature *(same for type 0, 2 and 3)*
Command: `cc d7 0e 00 02`


### Response (3 bytes)
| First byte | Last byte | Description                                            |
| ---------- | --------- | ------------------------------------------------------ |
|         0  |         1 | Temperature in 1/10 K, as little endian 16 bit integer |
|         2  |         2 | Always `06`                                            |


## Charge level (coulomb counter?)
Command: `cc d7 19 00 04`

### Response (5 bytes)
| First byte | Last byte | Description                                  |
| ---------- | --------- | -------------------------------------------- |
|         0  |         3 | Charge level as 32 bit little endian integer |
|         4  |         4 | Always `06`                                  |

The BTC04 divides this value 2880 and then by battery capacity in Ah.
