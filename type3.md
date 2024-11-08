# Battery type 3
If the battery fails the tests for type 0 and 2, but does respond correctly to `cc d4 2c 00 02`, it is type 3.

## Type 3 identifying command (unknown)
### Command
`cc d4 2c 00 02`

### Response

| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | ?           |
|         2  |         2 | Always `06` |



## Overdischarge count
### Command
`cc d6 09 03 01`
### Response (2 bytes)
| Byte | Description |
| ---- | ----------- |
|    0 | Integer count of overdischarge events |
|    1 | Always `06` |


## Health
### Command
`cc d6 38 02 02`
### Response (3 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | "Health" as 16 bit little endian integer |
|         2  |         2 | Always `06` |
BTC04 divides this value by battery capacity.


## Overload counters
### Command
`cc d6 5b 03 04`
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

## Temperature *(same for type 0, 2 and 3)*
### Command
`cc d7 0e 00 02`
### Response (3 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Temperature in 1/10 K, as little endian integer |
|         2  |         2 | Always `06` |
