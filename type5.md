# Battery type 5 (F0513 based)
If the value of byte 3 (counting from 0) of ROM id is less than 100, the battery is type 5.

## Read cell voltages
### Commands
`31` - Voltage for cell 1

`32` - Voltage for cell 2

`33` - Voltage for cell 3

`34` - Voltage for cell 4

`35` - Voltage for cell 5

### Response
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Cell voltage in millivolt. Little endian integer |



## Temperature
### Command
`cc 52`
### Response (2 bytes)
| First byte | Last byte | Description |
| ---------- | --------- | ----------- |
|         0  |         1 | Temperature in 1/10 K, as little endian integer |
