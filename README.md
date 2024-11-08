# Notes on how to communicate with Makita 18V batteries

Makita LXT 18V batteries feature some sort of battery management system, that keeps track of cell voltage, temperature, state of charge, overload, undervoltage. The yellow connector 

## Electrical connection

### Yellow connector
Proprietary 8 pin connector with 2.54 mm pitch. Both genders can be sourced from aliexpress.

| Pin | Description                                                                    |
| --- | -------------------------------------------------------------------------------|
|  1  |                                                                                |
|  2  | 1-Wire data                                                                    |
|  3  |                                                                                |
|  4  |                                                                                |
|  5  |                                                                                |
|  6  | Enable (active high)                                                           |
|  5  |                                                                                |
|  7  | *N/C*                                                                          |

Pin 1 is closest to negative battery terminal.

Both pin 2 and 6 are referenced to the negative battery terminal - the rest are probably as well.

Pin 6 must be pulled high (3.3V will do) before and during communication on pin 2. I don't know if it also serves as a open-collector output. It's probably safest to pull up through a 4.7 kiloohm resistor, and it seems to work well enough.

External pull-up on the 1-wire bus on pin 2 must be provided. A 4.7 kiloohm resistor to 3.3V seems to work. Pull-up to 5V should also work.

## 1-wire protocol
Standard 1-wire timing seems to work well enough.

Every command is preceeded by a 1-wire reset sequence, to which the battery asserts presence.

Commands that begin with `cc` can alternatively be preceeded by a rom id command  and read (send `33`, receive 8 bytes) instead of just sending `cc`.

## Battery message command
Command: One of
 * `cc aa 00`
 * `33 [read 8 bytes of rom id] aa 00`
 * `cc f0 00`
 * `33 [read 8 bytes of rom id] f0 00`

The BTC04 and the DC18RC uses the last form.

### Response (32 bytes)
Nybble oriented, with least significant nybble first, such that nybble 0 is the 4 LSB of byte 0.

| Nybble | Description                                                                                  |
| ------ | --------------------------------                                                             |
|  0..15 | ?                                                                                            |
| 16..21 | ?                                                                                            |
|     22 | Battery type, high 4 bits                                                                    |
|     23 | Battery type, low 4 bits                                                                     |
| 24..31 | ?                                                                                            |
|     32 | Capacity in 1/10Ah, high 4 bits                                                              |
|     33 | Capacity in 1/10Ah, low 4 bits                                                               |
|     34 | BL36xx check, high 4 bits                                                                    |
|     35 | BL36xx check, low 4 bits                                                                     |
|     36 | Some 16 bit value, must not be zero                                                          |
|     37 | Some 16 bit value, must not be zero                                                          |
|     38 | Some 16 bit value, must not be zero                                                          |
|     39 | Some 16 bit value, must not be zero                                                          |
|     40 | Failure code. 0=OK, 1=Overloaded, 5=Warning, 15=*TODO*. BMS considered failed if not 0 or 5. |
|     41 | Checksum of nybbles 0..15                                                                    |
|     42 | Checksum of nybbles 16..31                                                                   |
|     43 | Checksum of nybbles 32..40                                                                   |
|     44 | Bit 2 set on cell failure                                                                    |
|     45 | ?                                                                                            |
|     46 | Bits 1..3: Damage rating of sorts                                                            |
|     47 | ?                                                                                            |
|     48 | Damage thing A, high 4 bits                                                                  |
|     49 | Damage thing A, low 4 bits                                                                   |
|     50 | Overload, high 4 bits                                                                        |
|     51 | Overload, low 4 bits                                                                         |
|     52 | Bit 0: Cycle count, bit 12                                                                   |
|     53 | Cycle count, bits 8..11                                                                      |
|     54 | Cycle count, bits 4..7                                                                       |
|     55 | Cycle count, bits 0..3                                                                       |
| 56..61 | ?                                                                                            |
|     62 | Checksum of nybbles 44..47                                                                   |
|     63 | Checksum of nybbles 48..61                                                                   |

#### Battery type (nybbles 22..33)
8 bit integer B:

0 <= B < 13: 4 cell battery. BL14xx?

13 <= B < 30: 5 cell battery. BL18xx.

B = 20: 5 cell battery, but special somehow. 

B = 30: 10 cell battery. Probably BL36xx.


#### Capacity (nybbles 32..33)
8 bit integer in units of 1/10 Ah. 

Used as is by the BTC04 in various calculations, even though it's off by 0.2Ah for some batteries.


#### Checksums
The checksums are calculated like this:

```python
min(sum(nybbles), 0xff) & 0xf
```

That doesn't look like a good checksum function, and it might not be, but it seems to be used as one, by the BTC04:

If the checksums of nybbles 0..15, 16..31, and 32..40 doesn't match nybbles 41, 42, and 43, the battery is considered broken.

The checksums of nybbles 44.47 and 48..61 are calculated and checked, but doesn't seem to factor into anything displayed on the BTC04.

