---
title: SRAM
---

The SRAM is at location ```$70:0000``` to ```$70:07FF```.

Setting a breakpoint for when ```SRAM $70:0000``` is read, I get a break at ```$0D:FD21``` like this

```
$0D/FD21 BF 00 00 70 LDA $700000,x[$70:0000] A:0001 X:0000
$0D/FD25 9F 78 23 7E STA $7E2378,x[$7E:2378] A:0000 X:0000
$0D/FD29 E8          INX                     A:0000 X:0000
$0D/FD2A E8          INX                     A:0000 X:0001
$0D/FD2B E0 00 01    CPX #$0100              A:0000 X:0002
$0D/FD2E D0 F1       BNE $F1    [$FD21]      A:0000 X:0002
```

It reads 16 bits from ```SRAM $70:0000``` (plus X) and stores it at ```$7E:2378``` (plus X)
It then increases X by 2, and checks that it's not ```0x100``` (256 decimal) yet
and branches to the start and repeats.

Each loop X get increased by 2, so the second loop reads 16 bits from ```SRAM $70:0002``` and copies to ```$7E:237A```

---

I think programmers want to touch SRAM as little as possible. SRAM always (as I've seen) has a checksum somewhere in the data, to verify that the data is not corrupt. A dying battery could corrupt the values and make it so you've defeated 65535 teams or something like that. So we saw the code that copies from SRAM into internal ram once, and from there no more SRAM access. Because from there on, the game reads the saved player data from internal ram ```$7E:2378``` and not the cart anymore. When the game wants to save to SRAM, it does the checksum routine on the data at ```$7E:2378``` onwards, then uploads all the data to ```SRAM $70:0000```. So it reads and writes sram only in a few situations and all in one go.

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

So you can put whatever values into the .srm file and the game won't reject it. Unexpected values may cause the game/emulator to crash.
