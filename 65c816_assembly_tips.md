---
title: 65c816 Assembly Tips
---

The thing with the SNES is that the A register is more powerful than the X and Y. The XY regs are supposed to be used for counting or indexing. With that, the commands LDA and STA can do 24 bit addresses, however LDX/LDY, STX/STY, and even STZ can only do 16 bit addresses. So


```
	STZ $7E1E2E
```


Should only assemble into STZ $1E2E and use the current DB which is $00, resulting in a zero written to $001E2E ($7E1E2E mirror).

Also note, STZ will be 00 or 0000 depending on if A is 8 or 16 bit.

I figured it was easiest to use the 24 bit STA commands for readability even though they take 1 cycle longer than 16 bit STA commands.