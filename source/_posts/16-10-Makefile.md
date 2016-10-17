---
title: Makefile入门
toc: true
date: 2016-10-14 14:06:19
categories: C语言, 技术
tags: Makefile. C
---

在写Contiki_Makefile-prj的时候，想到可能有的人一点都不了解Makefile,所以写此文以供入门。

<!--more-->
# keil vVersion环境的编译过程
了解Makefile，主要是针对没有接触过linux的人，但是了解window下的单片机，dsp等开发环境。因此先简单说下window下的开发环境，以keil uVersion4为例。  
## 新建工程完成编译
新建工程/打开已有工程，我们常用的操作是在项目名称上“右键”->`Build Target`（或`Rebuild Target`），然后等待编译完成 ，输出hex文件或bin文件。  
那么这个`Build Target`执行了具体做了什么呢。我试了一下`Rebuild Target`：
```
Rebuild target 'Target 1'
compiling misc.c...
...此处省略很多行
compiling system_stm32f10x.c...
assembling startup_stm32f10x_hd.s...
compiling main.c...
compiling adc.c...
compiling adc_calc.c...
linking...
Program Size: Code=7812 RO-data=1312 RW-data=76 ZI-data=3676  
FromELF: creating hex file...
"ADC_TEST.axf" - 0 Error(s), 0 Warning(s).
```
可以看到只有：compiling、assembling、linking、elf转hex。简单来说就是：编译、链接。  
编译：就是将.c文件，.asm/.s文件编译，输出机器可识别的二进制文件.o文件；
链接：编译是把单个文件编译成一个个的.o文件，还不能直接给机器，因为程序中的包含、调用、变量的空间分配等还没有做好，也就是说这个二进制文件与处理器内的存储空间之间没有建立起一一对应的关系，所以就需要把所有的二进制文件一个一个连起来，把不同的代码放到不同的区域。所有相关的.o文件组合成一个新的文件.elf文件，不过通常需要的是hex，bin文件，所以看到了就是最后的FromELF,完成最终hex文件的创建。

## 编译过程-参数研究
不管是哪种平台这个编译、链接的过程是不会改变的。只是针对不同平台有不同的编译器，这个平台包括操作系统和处理器。
例如上面的keil，自带编译器，针对不同的处理器：51，cortex-M3\M4\A...，Arm7，Arm9等，因其机器内数据位宽不一致，指令集也有所差异，所以编译器处理的时候是有所去别的。为了一探究竟，下面研究下keil的配置界面参数：  

### C/C++参数
在`Project`窗口的项目根节点上右键选择`Option For Target`选项，进入C/C++选项页，最后有一项是`complier control string`，其中内容如下：
```
-c --cpu Cortex-M3 -g -O0 --apcs=interwork -I.\source\lib 
-I D:\Keil\ARM\RV31\INC 
-I D:\Keil\ARM\CMSIS\Include 
-I D:\Keil\ARM\Inc\ST\STM32F10x 
-o "*.o" --omf_browse "*.crf" --depend "*.d"
```
其中-I是用来描述包含路径的，第一行的`-I`后面的路径是在C/C++选项页的Include Paths栏所显示的内容。
第一行指定CPU等信息，最后一行指定编译输出文件信息。

### ASM参数
同样在Asm选项页的`Assembler control string`有如下内容：
```
-cpu Cortex-M3 -g --apcs=interwork 
-I D:\Keil\ARM\RV31\INC 
-I D:\Keil\ARM\CMSIS\Include 
-I D:\Keil\ARM\Inc\ST\STM32F10x 
--list "*.lst" --xref -o "*.o" --depend "*.d"
```
这里主要描述将.o文件链接输出为可执行文件。

简单点说这个配置界面的参数就是告诉编译器以什么样的规则编译源代码。  

那么能不能把这些配置信息给提取出来放置到一个专门的文件中去呢，然后执行某个命令/操作，自动完成编译、链接。答案当然是可以的。
并且这个可以并不区分windows或linux。

# 采用Makefile的helloworld函数
还记得上面提到的编译、链接吗，我们使用两个命令代替：`make`、`link`。同时呢，摆脱开发环境keil的限制，需要安装GCC工具，这个gcc就是可以将C文件变化成二进制文件的工具，叫编译器或工具链。

## GCC工具安装
安装完成后，windows下`win+R`打开终端命令行，输入`make -v`，查看版本信息：
```
C:\Users\Administrator>make -v
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-pc-mingw32
```
## 源文件和Makefile
ok，有了编译工具了，那么下面就写个hello函数进行测试吧。
```
//hello.c
#include "stdio.h"

int main (void)
{
    printf ("hello!");
    return 0;
}
```
```
#Makefile
all: hello
hello: hello.c
#gcc前面是tab，不是空格，Makefile中的命令前需要有tab，如果编辑器会把tab转换成空格是不可以的。
	gcc -o hello hello.c
clean:
#rm前面是tab，不是空格
	rm -rf *.o
```
## 编译、运行
这里有了源文件，有了Makefile，有个编译工具，下面就看看怎么了编译运行吧。
windows下`win+r`，输入`cmd`开命令行,找到并进入hello.c和Makefile所在工程目录。  
命令行下使用  
* cd name打开某个文件夹，输入文件夹名字时用tab键可自动补全；
* cd ..返回上一级文件夹；
* e:直接进入E盘根目录。
* dir 显示当前文件夹下的文件或文件夹列表

进入工程目录后，执行`make`命令,得到以下结果：
```
$ F:\make-hello>make
$ gcc -o hello hello.c
```
执行`dir`命令，发下文件夹中多了个hello.exe
```
$ dir
Makefile  hello.c  hello.exe
```
命令行输入`hello.exe`/`hello`执行输出文件，可得到如下结果：
```
F:\make-hello>hello
hello!
```
到这里，我们完成了一个简单函数的编译、运行。没有使用集成开发环境。  

## Makefile里做了什么
```
all: hello
hello: hello.c
	gcc -o hello hello.c
clean:
	rm -rf *.o
```
Makefile中的`all`是个伪目标，make命令执行的时候会先来找到all，之后找到all中的hello。hello是个目标，它依赖于后面的hello.c，这个目标中包含命令行`gcc -o hello hello.c`。  
这里的`-o`是不是很像keil中的`-o "*.o" --omf_browse "*.crf" --depend "*.d"`。这个`o`是output的缩写，用于设定输出信息的。而前面的gcc就是编译命令了，执行的make来触发这个命令。如果有多个文件相互依赖的话还会触发link操作。

如果想更深入的看懂理解Makefile，需要去学习Makefile的语法，命令，函数。

***
