---
title: SRAM
---

## Where is it?
Before we dive into memory addresses, we want to know what kind of ROM it is. Using **Geiger's Snes9x**, `Load Game`, then click `ROM Information...` you should see
```
Name: NBA JAM TOURNAMENT ED0
Speed: 30/FastROM
ROM Map: LoROM
Type: 02
Kart contents: ROM+RAM+BAT
Header ROM Size: 32Mbits
Calculated ROM Size: 24 Mbits
SRAM size: 2KB (16Kbit)
Actual Checksum: AFD2
Header Checksum: AFD2
Header Checksum Compliment: 502D
Output: NTSC 60Hz
CRC32:	1FBC1DDB
Licensee: Acclaim
ROM Version: 1.0
Region: USA/Canada
```

The things to note here are **ROM Map** and **Speed**. Typically you'll see **LoROM** and **HiROM**. This allows games to use different memory layouts. Here we're using **LoROM**. Using the document **SNES Memory Mapping** in the [Documents](tools_docs_links.html#documents) section, you'll see under **Mode 20: LoROM Memory Model**

```
$70-$77 | $0000-$7FFF | (Mode 20 SRAM) 256KBytes
```

So we can access a byte or word in SRAM at `$70:0000`, and through mirrors, that same byte or word can be read or written at `$77:0000`, or `$73:0000` since those are mirrors. Remember, these are addresses for SRAM memory in the cartridge, not ROM, and not internal SNES memory.

---

The other thing to know is that we're working with a **FastROM**, it uses the memory addresses `80:8000 - FF:FFFF` in **Lunar Address** opposed to `00:8000 - 6F:FFFF`. This is when accessing ROM addresses. This is how to decide which address to use in *Lunar Address*.

> The SNES lets you access ROM through bank $00 onwards and bank $80 onwards such that locations $008000 and $808000, $008001 and $808001, $008002 and $808002 etc... all access the same locations. When accessing bank $00 onwards the 65816 runs at 2.68Mhz. However, when accessing bank $80 onwards the 65816 can run at 2.68Mhz or 3.58Mhz depending on how you set bit 0 of $420D.

So **FastROM** is `80:8000`, **SlowROM** is `00:8000`.

---

## Format
```
The saveram consists of 256 bytes

Four blank bytes and a string
.db 0, 0, 0, 0
.db "Snakes BBU Area/NBA JAM TE"

Then 16 save slots, each slot is 12 bytes
.db 0x0B,0x0E,0x0B,	;initials "LOL"
.db 0			;gets set to zero as part of initials
.db 0x1D		;wins
.db 0			;losses
.db 0x1D		;streak
.db 1			;file index?
.db 0x1B		;27 teams defeated
.db 0			;?
.db 0			;?
.db 0			;?

Then 16 more bytes, checksums per slot?

.db 0x19, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF
.db 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF

Then 2 checksum words

.dw 0xA4A4
.dw 0x561E

Then a string and four blank bytes
.db "*Claire R*"
.db 0, 0, 0, 0
```

---

## Internal memory copy
The SRAM is at location `$70:0000` to `$70:07FF`.

Setting a breakpoint for when `SRAM $70:0000` is read, I get a break at `$0D:FD21` like this

```
$0D/FD21 BF 00 00 70 LDA $700000,x[$70:0000] A:0001 X:0000
$0D/FD25 9F 78 23 7E STA $7E2378,x[$7E:2378] A:0000 X:0000
$0D/FD29 E8          INX                     A:0000 X:0000
$0D/FD2A E8          INX                     A:0000 X:0001
$0D/FD2B E0 00 01    CPX #$0100              A:0000 X:0002
$0D/FD2E D0 F1       BNE $F1    [$FD21]      A:0000 X:0002
```

It reads **16 bits** from `SRAM $70:0000` (plus `X`) and stores it at `$7E:2378` (plus `X`)
It then increases `X` by **2**, and checks that it's not **0x100** (**256 decimal**) yet
and branches to the start and repeats.

Each loop `X` get increased by **2**, so the second loop reads **16 bits** from `SRAM $70:0002` and copies to `$7E:237A`

---

## Skip sanity check
I think programmers want to touch SRAM as little as possible. SRAM always (as I've seen) has a checksum somewhere in the data, to verify that the data is not corrupt. A dying battery could corrupt the values and make it so you've defeated **65,535** teams or something like that. So we saw the code that copies from SRAM into internal ram once, and from there no more SRAM access. Because from there on, the game reads the saved player data from internal ram `$7E:2378` and not the cart anymore. When the game wants to save to SRAM, it does the checksum routine on the data at `$7E:2378` onwards, then uploads all the data to `SRAM $70:0000`. So it reads and writes SRAM only in a few situations and all in one go.

I did find this, it ignores the two(?) checksums.

```
;==================
;skip saveram check
;==================
org $0DFAF7
	NOP #2

org $0DFAD9
	NOP #2
```

So you can put whatever values into the .srm file and the game won't reject it. **Unexpected values may cause the game/emulator to crash.**
