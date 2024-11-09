# Battery type 3
If the battery fails the tests for type 0 and 2, but does respond correctly to `cc d4 2c 00 02`, it is type 3.

## Type 3 identifying command (unknown)
Command: `cc d4 2c 00 02`

### Response
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | ?           |
|         2  |         2 | Always `06` |



## Overdischarge count
Command: `cc d6 09 03 01`
### Response (2 bytes)
| Byte | Description |
| ---- | ----------- |
|    0 | Integer count of overdischarge events |
|    1 | Always `06` |


## Overload counters
Command: `cc d6 5b 03 04`
### Response (6 bytes)
| Byte | Description     |
| ---- | --------------- |
|    0 | Counter A       |
|    1 | ?               |
|    2 | Counter B       |
|    3 | Counter C       |
|    4 | ?               |
|    6 | Always `06`     |

In the BTC04, the 3 counters are added together. They might count different types of overload events.
Counter C is stored separately though.

Overload percentage is calculated the same way as for type0.


## Health
Command: `cc d6 38 02 02`
### Response (3 bytes)
| First byte | Last byte | Description                              |
| ---------- | --------- | ---------------------------------------- |
|         0  |         1 | "Health" as 16 bit little endian integer |
|         2  |         2 | Always `06`                              |

Health on a scale from 0 to 4 is calculated in the same way as for type0.


## Temperature *(same for type 0, 2 and 3)*
Command: `cc d7 0e 00 02`
### Response (3 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Temperature in 1/10 K, as little endian integer |
|         2  |         2 | Always `06` |


## Voltages *(same for type 0, 2 and 3)*
Command: `cc d7 00 00 0c`
### Response (13 bytes)
| First byte | Last byte | Description                                     |
| ---------- | --------- | ----------------------------------------------- |
|         0  |         1 | Pack voltage, as 16 bit little endian integer   |
|         2  |         3 | Cell 1 voltage, as 16 bit little endian integer |
|         4  |         5 | Cell 2 voltage, as 16 bit little endian integer |
|         6  |         7 | Cell 3 voltage, as 16 bit little endian integer |
|         8  |         9 | Cell 4 voltage, as 16 bit little endian integer |
|        10  |        11 | Cell 5 voltage, as 16 bit little endian integer |
|        12  |        12 | Always `06`                                     |

All voltages are given in millivolt.

