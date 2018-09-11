---
title: 65c816 Assembly Tips
---

The thing with the SNES is that the ```A``` register is more powerful than the ```X``` and ```Y```. The ```XY``` regs are supposed to be used for counting or indexing. With that, the commands ```LDA``` and ```STA``` can do **24 bit** addresses, however ```LDX```/```LDY```, ```STX```/```STY```, and even ```STZ``` can only do **16 bit** addresses. So


```
	STZ $7E1E2E
```


Should only assemble into ```STZ $1E2E``` and use the current ```DB``` which is $00, resulting in a zero written to ```$00:1E2E``` (```$7E:1E2E``` mirror).

Also note, ```STZ``` will be **$00** or **$0000** depending on if ```A``` is **8 bit** or **16 bit**.

I figured it was easiest to use the **24 bit** ```STA``` commands for readability even though they take 1 cycle longer than **16 bit** ```STA``` commands.

---

Below is an example of how to change the bank. Typically you want to push databank (```PHB```, save it), then load an **8 bit** value into ```A```, ```X```, or ```Y```, then push the value. Then pull it into the **data bank** (```PLB```). Once our code is finished, we pull the original bank with another ```PLB``` and the original code can continue.

```
change_bank_to_7F:
	PHB						;save bank because we're about to change it

	SEP #$20					;set 8 bit A processor settings
	LDA #$7F					;bank for our brand new variables (8 bit constant)
	PHA						;push it (8 bit push)
	REP #$20					;set 16 bit A processor settings

	PLB						;pull it to bank (this is always 8 bit)
```