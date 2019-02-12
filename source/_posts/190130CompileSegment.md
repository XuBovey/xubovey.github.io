---
title: 编译结果各字段含义
toc: true
date: 2019-01-30 21:02:29
categories:
tags:
---

[参考](https://mcuoneclipse.com/2013/04/14/text-data-and-bss-code-and-data-size-explained/)

```
   text       data        bss        dec        hex    filename
 0x1408       0x18      0x81c       7228       1c3c    size.elf
```

text
‘text’ is what ends up in FLASH memory. I can show this with adding

data
‘data’ is used for initialized data. This is best explained with the following (global/extern) variable:

bss
The ‘bss’ contains all the uninitalized data.

💡 bss (or .bss, or BSS) is the abbreviation for ‘Block Started by Symbol’ by an old assembler (see this link).

dec
The ‘dec’ (as a decimal number) is the sum of text, data and bss:   
```
dec = text + data + bss
```

Size – GNU Utility
The size (or printsize) GNU utility has more options:

Summary
I hope I have sorted out things in a correct way. The way how the initialized data is reported might be confusing. But with the right knowledge (and .map file in mind), things get much clearer:

‘text’ is my code, vector table plus constants.

‘data’ is for initialized variables, and it counts for RAM and FLASH. The linker allocates the data in FLASH which then is copied from ROM to RAM in the startup code.

‘bss’ is for the uninitialized data in RAM which is initialized with zero in the startup code.

