---
title: ROM expansion
---

## Making more room
If you open up the original NBA Jam TE ROM in your hex editor, and go all the way to the end, you'll see the last byte is at `0x2FFFFF`. This game is **24 megabit**, which is **3,145,728 bytes**, but we want more room for graphics and code so we'll expand it to **32 megabit**. If you go do the same thing to a **32 megabit** game (**4,194,304 bytes**) you'll see the last byte is at `0x3FFFFF`. Now you can just fill the space in the NBA Jam TE ROM with a hex editor until you reach that size, or even use a ROM expansion program. I like to use a built in feature in **xkas** and simply place a byte at that location. When assembling, **xkas** will then fill the space to that byte at the end.

```
org $FFFFFF			;pad to 32 mbit
	db $00
```

Wait how did we get SNES memory address **$FF:FFFF** from hex editor file address **0x3FFFFF**?

Open up **Lunar Address**, make sure it's set to *LoROM - PC 80:8000 - FF:FFFF* and type **3FFFFF** into the left where the *PC File Address* goes. You will then see *SNES LoROM $FF:FFFF* which is the last SNES memory address that can accessed in a **32 megabit** game. ROM file locations are not the same as SNES memory locations, so we need to convert them with **Lunar Address**. When we write code, **xkas** only deals in SNES memory locations.

---

So once we've expanded our ROM, where do we start adding stuff? We will start adding stuff immediately at the end of the original ROM, right after the **24 megabit** ends. So if the game ended at file location `0x2FFFFF`, we will put data into the file starting at location `0x300000`. So type `300000` into *PC File Address* and you get *SNES LoROM* `$E0:8000`. Always use the *PC File Address* and let **Lunar Address** calculate the SNES memory address if you have a ROM location. `$E0:8000` is where we can begin adding data and code to the ROM.

If you typed in *SNES LoROM $E0:0000*, you'd get *PC File Address 0x300000* which looks OK, but that SNES address is actually wrong. Do it the other way around, enter *PC File Address 30:0000* first and you'll get the correct *SNES LoROM $E0:**8**000*. That **8** makes a difference.
