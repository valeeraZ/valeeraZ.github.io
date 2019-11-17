---
title: 单核心处理器，总线架构及外围设备
layout: post
subtitle: MIPS
date:       2019-11-17
author:     "Zhao"
header-img: "img/system.png"
tags: 
    - System
---

**版权声明**  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/80x15.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/">知识共享署名-非商业性使用-相同方式共享 3.0 未本地化版本许可协议</a>进行许可。

![almo.jpg](https://i.loli.net/2019/11/17/uMTHljYrGtcd5L8.jpg)

<!-- TOC -->

- [前言](#前言)
- [TTY](#tty)
- [MIPS虚拟地址内存映射](#mips虚拟地址内存映射)
- [操作系统](#操作系统)
    - [软件架构](#软件架构)
    - [具体应用](#具体应用)
    - [实例](#实例)
- [SR (Status Register $12)](#sr-status-register-12)
- [CR (Cause Register $13)](#cr-cause-register-13)
- [进入内核的步骤](#进入内核的步骤)
    - [syscall](#syscall)
    - [Exception异常](#exception异常)
- [从内核返回用户应用](#从内核返回用户应用)

<!-- /TOC -->

# 1. 前言

本篇文章我们将学习计算机物理，寻址空间及软件组成原理，探究处理器，内存以及外围设备间的信息沟通。如下图所示，我们所研究的内容包含：
- 一枚MIPS32架构的处理器，包含一级缓存L1用来缓存数据和指令
- 二级缓存L2，用来与外部内存RAM链接
- 只读存储器ROM，用来存放机器启动时的代码
- 用来控制终端读写的外围设备（即TTY屏幕和键盘）
- 一根总线，连接一个主设备和三个目标设备

![archi.jpg](https://i.loli.net/2019/11/17/nBisbQpDJI3xCaV.jpg)



系统在处理指令时，使用小型的开发系统GIET来处理异常（法语：Gestionnaire d’Interruptions, Exceptions et Trappes）（翻译：GIET : Interruption / Exception / Trap Handler for MIPS32 processor）。

# 2. TTY

从历史上看，终端刚开始就是终端机，配有打印机，键盘，带有一个串口，通过串口传送数据到主机端，然后主机处理完交给终端打印出来。电传打字机(teletype)可以被看作是这类设备的统称，因此终端也被简称为 TTY(teletype 的缩写)。  
后来的终端慢慢演变成了键盘 + 显示器。如果我们要把内容输出到显示器，只要把这些内容写入到显示器对应的 TTY 设备就可以了，然后由 TTY 层负责匹配合适的驱动完成输出。  
每个外围设备所对应的寄存器的地址会映射在内存中。
每个终端与四个寄存器相连
- `TTY_WRITE`：将一个ascii字符显示在终端的屏幕上，处理器需要使用`sb`指令将一个byte写入`TTY_WRITE`寄存器的地址
- `TTY_READ`：从键盘读取一个ascii字符，处理器需要使用`lb`指令将一个byte写入`TTY_READ`寄存器的地址
- `TTY_STATUS`：判断`TTY_READ`中有无字符，处理器需要从`TTY_STATUS`读取一个长度为32bits的词
    - 若这个词第0位不为0，则寄存器`TTY_READ`不为空
    - 若这个词第1位不为0，则寄存器`TTY_WRITE`不为空
- `TTY_CONFIG`：软件可以将外围设备的信息写入在这个寄存器内

若`TTY_WRITE`收到读取信息的指令，或者`TTY_READ`/`TTY_STATUS`收到写入信息的指令，则TTY控制器会返回错误代码，并在处理器端触发一个异常（exception）：错误的程序将停止运行（interruption），处理器将会跳转至异常处理系统以触发异常*Data BUS Error*。  
TTY控制器能够作用于多个终端，而每个终端都具有4种TTY的控制权。每个TTY地址长度为4字节，因此每个终端也就占据4*4=16字节的地址长度。

![périphériques cibles.jpg](https://i.loli.net/2019/11/17/PZgNbf86LWD2pBY.jpg)

下图记录了两个终端持有8个TTY寄存器的32bits地址段，其中
- 第2，3位记录终端
- 第4，5位记录4个寄存器中的哪一个

![tty_base.jpg](https://i.loli.net/2019/11/17/z7bdvIGuMxjDJQc.jpg)

当MIPS做出load或store指令时，所有的组件都会受到目标地址。每个组件在虚拟地址映射所属的地址段（segment de l'espace d'adressage）：若映射到的地址属于此组件管理，则此组件处理此请求；否则则不做任何事。

# 3. MIPS虚拟地址内存映射

![espace adressage.jpg](https://i.loli.net/2019/11/17/ef6XTA12j3vurBg.jpg)

用户模式地址区：
- `seg_code`：用户程序（在`main.c`里定义）以及一些用户模式下
所使用的外围设备的系统服务函数（在`stdio.c`中）。对应的地址为`seg_code_base = 0x00400000` 
- `seg_data`：用户程序使用的全局数据，对应的地址是`seg_data_base = 0x10000000`
- `seg_stack`：程序所使用的栈，对应的地址是
`seg_stack_base = 0x20000000`  

内核模式地址区：
- `seg_reset`：在ROM中。机器启动代码，对应的地址是`seg_reset_base = 0xBFC00000`，代码定义在文件`reset.s`
- `seg_kcode`：受操作系统保护的代码区。对应的地址是`seg_kcode_base = 0x80000000`，主要包含了**GIET**代码
- `seg_kdata`：受操作系统保护的数据，对应的地址`seg_kdata_base = 0x81000000`
- `seg_kunc`：操作系统中**不可缓存**的数据，对应的地址是`seg_kunc_base = 0x82000000`
- `seg_tty`：包含了四个TTY，对应的地址为`seg_tty_base = 0x90000000`  

The base address of the segment containing this code MUST be `0x80000000` , in order to have the entry point at address `0x80000180` !  
GIET系统地址起始点为`0x80000000`，其中进入点为`0x80000180`。多出来的`0x180`为`.space 0x180`所定义的384字节的空间，这一段空间留空。（详见[giet.s](https://www-soc.lip6.fr/trac/sesi-almo/chrome/site/docs/ALMO-giet-src-code.pdf)源代码）  

GIET代码启动时存储在RAM中因为其代码属于操作系统一部分，在boot阶段运行。GIET数据存储在`seg_kdata`和`seg_kunc`中。

# 4. 操作系统

1. 管理硬件及应用资源
    - 对硬件的调用：终端，硬盘，网络等
    - 内存分配
    - 处理器资源分配
2. 管理外围设备事件
    - 结束对硬件的调用
    - 处理周期性操作
3. 处理程序异常
    - 以0做除数
    - 非法指令
    - 分页错误，溢出等

总结：操作系统应保证：
- 服务质量（Quality of Service）：公平，优先权，期限
- 应用安全：机密性，整体性
- 硬件使用可靠性

## 4.1. 软件架构

![archi logicielle.jpg](https://i.loli.net/2019/11/18/TQCNeZqjrMEHgBi.jpg)

- 上部：应用层，可以直接通向某些地址段，但若要接触硬件层（下部matériel）则需要通过系统代码库（biblithèques système）
- 系统代码库（biblithèques système）：定义了需要使用的API。这些功能可在用户模式下运行，通过调用系统内核服务来进行（`syscall`）。服务通过`eret`（异常返回）返回给用户信息。
- 中部：系统内核，他接收来自应用层的服务请求（`syscall`）
- 下部：硬件，通过各种load/store指令写入数据

## 4.2. 具体应用

一个程序应该至少有一个`main()`函数用来启动，他的代码，数据以及堆栈都存储在内存中只要这个程序是可以在用户模式下运行的（也就是地址小于`0x80000000`）。  
当应用需要访问外围设备时，他需要向系统内核通过`syscall`发出请求，通过系统代码库中的某个API来实现。  
```
$4 à $7 ⇐ 系统服务参数
$2 ⇐ 系统服务代号
syscall ≃ jal kernel
$2 ⇒ 返回错误代码，或0表示成功
```

## 4.3. 实例
```c
#include <stdio.h>
__attribute__((constructor)) void main(void)
{
    char byte;
    char str[] = "\nHello World!\n";
    while (1) {
        tty_puts(str);
        tty_getc(&byte);
        if (byte == 'q') {
            exit(0);
        }
    }
    exit(0);
}
```

这段程序为一个死循环，在每次循环中在屏幕上显示Hello World（`tty_puts(str);`），并且从键盘读入一个字符（`tty_getc(&byte);`），当读取到的字符为q时，退出循环，程序结束。

输出信息的步骤：
1. 启动时，处理器跳转到`0xBFC00000`并运行存储在`seg_reset`的启动代码
2. 当处理器收到eret返回值时（即机器启动结束），开始运行`main()`函数的第一行代码（存储在`seg_code`）中
3. 当处理器运行至`tty_puts(str);`时（存储在`seg_code`），跳转到其所在的地址段
4. 处理器运行在`tty_puts()`中的`syscall`跳转至`0x80000180(seg_kcode)` 
5. **GIET**开始处理第一次跳转，调用他本身的系统函数`__tty_write()`，以在屏幕上输出一行长度已知的字符串。这个函数从`seg_data`中依次读取字符，并将其写入`seg_tty`中。（即在寄存器`TTY_WRITE`中）
6. `__tty_write()`函数返回至`tty_puts()`再返回至`main()`

# 5. SR (Status Register $12)

![SR.jpg](https://i.loli.net/2019/11/18/uLmz7HZJxNXYTEW.jpg)

1. 第15 - 第8位：IM[7:0] Interrupt Mask即IRQ。高六位为Hardware，低两位为Software
2. UM：User Mode 若为1，则为USER模式
3. ERL：Error Level
4. EXL：Exception Level
5. IE：Interrupt Level 若为1，则存在中断

若ERL+EXL=1，则在KERNEL模式，IRQ非激活态

典型值：
- 0xFF11：用户模式
- 0xFF13：在内核
- 0xFF01：syscall
- 0xFF00：临界区段

# 6. CR (Cause Register $13)

![CR.jpg](https://i.loli.net/2019/11/18/bF5KVMIgOokisft.jpg)

仅需要注意XCODE数段，4位16个可能至代表系统进入内核模式的原因

# 7. 进入内核的步骤

## 7.1. syscall

1. EPC <-PC : 保存syscall的地址
2. SR[EXL] = 1
3. CR[XCODE] = 8 即1000 SYS，为进入的原因
4. PC = 0x80000180

## 7.2. Exception异常

1. EPC <-PC : 保存syscall的地址
2. SR[EXL] = 1
3. CR[XCODE] = 异常原因
4. PC = 0x80000180

# 8. 从内核返回用户应用

1. SR[EXL] = 0
2. PC = EPC





