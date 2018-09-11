---
title: 65c816 Assembly Tips
---

## Accumulator
The thing with the SNES is that the ```A``` register is more powerful than the ```X``` and ```Y```. The ```XY``` regs are supposed to be used for counting or indexing. With that, the commands ```LDA``` and ```STA``` can do **16 bit** or **24 bit** addresses, however ```LDX```/```LDY```, ```STX```/```STY```, and even ```STZ``` can only do **16 bit** addresses. So


```
	STZ $7E1E2E
```


Should only assemble into ```STZ $1E2E``` and use the current ```DB``` which is $00, resulting in a zero written to ```$00:1E2E``` (```$7E:1E2E``` mirror).

Also note, ```STZ``` will be **$00** or **$0000** depending on if ```A``` is **8 bit** or **16 bit**.

I figured it was easiest to use the **24 bit** ```STA``` commands for readability even though they take 1 cycle longer than **16 bit** ```STA``` commands.

---

## Data Bank
The Data Bank register makes a **24 bit** address out of a **16 bit** address. The SNES CPU simply adds this byte to a **16 bit** address. So if you want to put a zero at ```$7F:1234```, you would have to make sure the ```DB``` is $7F, and then

```
	STZ $1234
```


Below is an example of how to change the bank. Typically you want to push databank (```PHB```, save it), then load an **8 bit** value into ```A```, ```X```, or ```Y```, then push the value. Then pull it into the **data bank** (```PLB```). Once our code is finished, we pull the original bank with another ```PLB``` and the original code can continue.

```
change_bank_to_7F:
	PHB				;save bank because we're about to change it

	SEP #$20			;set 8 bit A processor settings
	LDA #$7F			;bank for our brand new variables (8 bit constant)
	PHA				;push it (8 bit push)
	REP #$20			;set 16 bit A processor settings

	PLB				;pull it to bank (this is always 8 bit)
```

---

## All Hell BRKs loose

Say we have the code below and assemble it with xkas

```
	CPY #$00B4
```

Then we run it in Geiger's Snes9x and the emulator crashes. When that happens, it's usually an **8 bit** vs. **16 bit** constant problem. We gotta watch out for **8 bit** or **16 bit** modes (it's a bitch). Below we see something's totally wrong.

```
$E0/005E C0 B4       CPY #$B4                A:0002 X:0000 Y:00B4
$E0/0060 00 F0       BRK #$F0                A:0002 X:0000 Y:00B4
```

We saw earlier that ```X``` and ```Y``` are **8 bit**, yet our code when assembled uses a **16 bit** immediate value (xkas has no idea what state the CPU will be in when the code is being inserted), therefore, the extra **00** then spills into the next command (which should be ```BEQ #$03 - 0xF0 0x03```) becomming ```BRK #$F0 - 0x00 0xF0```. So the fix, is use an **8 bit** immediate value in our source, no problemo. Xkas does have an **ASSUME** directive which can help this issue.

```
	CPY #$B4
```
