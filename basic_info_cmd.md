## Basic battery information command
As far as I know, all batteries support this command, and all return data in the same annoying format.

Command: One of
 * `cc aa 00`
 * `33 [read 8 bytes of rom id] aa 00`
 * `cc f0 00`
 * `33 [read 8 bytes of rom id] f0 00`

The BTC04 and the DC18RC uses the last form.

Note that repeating the command in a form that ends in `f0 00` will change some state, and cause the battery to stop responding. This state is cleared by pulling the enable pin low for a short while.

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
|     34 | Flags, high 4 bits                                                                           |
|     35 | Flags, low 4 bits                                                                            |
|     36 | Some 16 bit value, must not be zero                                                          |
|     37 | Some 16 bit value, must not be zero                                                          |
|     38 | Some 16 bit value, must not be zero                                                          |
|     39 | Some 16 bit value, must not be zero                                                          |
|     40 | Failure code. 0=OK, 1=Overloaded, 5=Warning, 15=*TODO*. BMS considered dead if not 0 or 5.   |
|     41 | Checksum of nybbles 0..15  (battery locked if not matching)                                  |
|     42 | Checksum of nybbles 16..31 (battery locked if not matching)                                  |
|     43 | Checksum of nybbles 32..40 (battery locked if not matching)                                  |
|     44 | Bit 2 set on cell failure                                                                    |
|     45 | ?                                                                                            |
|     46 | Bits 1..3: Damage rating for old batteries                                                   |
|     47 | ?                                                                                            |
|     48 | Overdischarge, high 4 bits                                                                   |
|     49 | Overdischarge, low 4 bits                                                                    |
|     50 | Overload, high 4 bits                                                                        |
|     51 | Overload, low 4 bits                                                                         |
|     52 | Bit 0: Cycle count, bit 12                                                                   |
|     53 | Cycle count, bits 8..11                                                                      |
|     54 | Cycle count, bits 4..7                                                                       |
|     55 | Cycle count, bits 0..3                                                                       |
| 56..61 | ?                                                                                            |
|     62 | Checksum of nybbles 44..47                                                                   |
|     63 | Checksum of nybbles 48..61                                                                   |


#### Checksums / locked state
The checksums are calculated like this:

```python
min(sum(nybbles), 0xff) & 0xf
```

That doesn't look like a good checksum function, and it might not be, but it seems to be used as one by the BTC04:

If the checksums of nybbles 0..15, 16..31, and 32..40 doesn't match nybbles 41, 42, and 43 respectively, the battery is considered broken (locked?). The battery is also considered broken, if nybbles 40..43 equals 0xffff.

If the battery is put into *test mode*, some of the checksums will fail, and the battery is in fact locked (no tool power, won't charge). Exiting test mode restores functionality, and checksums match again.

The checksums of nybbles 44..47 and 48..61 are calculated and checked, but doesn't seem to factor into anything displayed on the BTC04.



#### Battery type (nybbles 22..33)
8 bit integer B:

0 <= B < 13: 4 cell battery. BL14xx?

13 <= B < 30: 5 cell battery. BL18xx.

B = 20: 5 cell battery, but special somehow. *TODO*

B = 30: 10 cell battery. Probably BL36xx.

Observed values in BL18xx batteries: 18 and 20.


#### Flags (nybbles 34, 35)
Probably flags.

The battery is type 6 (a 10 cell non-xgt battery, likely BL36xx) if the value is 0x1e = 0b0001_1110.

Observed values in BL18xx batteries: 0x0d and 0x0e.


#### Capacity (nybbles 32, 33)
8 bit integer.

Looks very much like batter capacity in units of 1/10 Ah, even though it's off by 10% for some batteries.

Used as is (the entire 8 bit integer) by BTC04 in various calculations.

Some observed values in BL18xx batteries:

 * BL1815N: 15
 * BL1830B: 28 or 30
 * BL1840: 36 or 40
 * BL1850B: 50 or 52


#### Damage rating (nybble 46, 3 MSB)
3 bit integer.

Possibly only applicable for very old battery types.

For some batteries, the BTC04 will resort to convert this value to the 0-4 health rating.

For those batteries, a damage rating below 3 corresponds to 4/4 health, 7 corresponds to 0/4 health, etc.


#### Overdischarge (nybbles 48, 49) 
8 bit integer.

For type5 and type6 batteries, BTC04 calculates overdischarge percentage *p* as follows:
```python
p = -5*x + 160
```

Also used in battery health calculation by the BTC04.

Must be non-zero for the battery to be of type 0.

Some observed values in BL18xx batteries: 27, 30, 31, 32, 33


#### Overload (nybbles 50, 51)
8 bit integer.

Probably only applicable for type5 or type6 batteries.

Used in battery health calculation by the BTC04 for batteries NOT of types 0, 2 or 3.

Must be non-zero for the battery to be of type 0.

Some observed values in BL18xx batteries: 26, 30, 31, 32, 34

For type5 and type6 batteries, BTC04 calculates overload percentage as follows:

```python
p = 5*x - 160
```


##### Health calculation for type5 and type6
For batteries of type 5 (F0513 based) or type6, BTC04 calculates *h*, its health rating on a scale from 0 to 4, from the above raw values for capacity, overdischarge, and overload:

```python
f_ol = max(overload - 29, 0)
f_od = max(35 - overdischarge, 0)
dmg = cycles + cycles * (f_ol + f_od) / 32
scale = 1000 if capacity in (26, 28, 40, 50) else 600
h = 4 - dmg / scale
```

