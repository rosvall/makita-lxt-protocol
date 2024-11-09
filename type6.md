# Battery type 6 (10 cell battery, likely BL36xx)
If byte 17 (counting from 0) of the response to the basic battery information command `cc aa 00` equals 30 (decimal), the battery is type 6.

## Enter state to read out voltages
Command: `cc 10 21`
### Response (0 bytes)
None

## Read cell voltages
Command: `d4`

### Response (20 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Scaled and offset cell 1 voltage. Little endian integer |
|         2  |         3 | Same for cell 2 |
|         4  |         5 | Same for cell 3 |
|         6  |         7 | Same for cell 4 |
|         8  |         9 | Same for cell 5 |
|        10  |        11 | Same for cell 6 |
|        12  |        13 | Same for cell 7 |
|        14  |        15 | Same for cell 8 |
|        16  |        17 | Same for cell 9 |
|        18  |        19 | Same for cell 10 |

Use the formula 

```python
v = 6000 - x/10
```

to convert to millivolt.

BTC04 uses the minimum cell voltage to calculate battery pack charge percentage.


## Temperature
Command: `d2`
### Response (1 byte)
Temperature as single byte integer.

Convert to degrees celcius with:

```python
t = (-40*x + 9323)/100
```
