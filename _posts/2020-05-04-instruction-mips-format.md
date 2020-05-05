---
title: MIPS指令格式
layout: post
subtitle: MIPS R I J instructions' formats
date:       2020-05-04
author:     "Zhao"
header-img: "img/system.png"
tags: 
    - System
---

本篇文章介绍mips汇编语言的三种指令格式。请确保已阅读上一篇文章[mips汇编语言入门](https://valeeraz.github.io/2019/09/28/architecture-mips/)。

# 汇编语言与机器语言

机器指令是由0,1的二进制码构成的因而人类无法阅读。相对应地，汇编语言允许我们使用相应的代码来编写指令。下面是对应的机器码和汇编语言：

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/Screenshot%202020-05-04%20at%2017.24.00.png?token=AJLBRHCFS2TDFD27ZKMRUKS6WA7LK)

编译器将高级别编程语言（例如C）编译为汇编语言，汇编语言编译器再讲汇编语言编译为机器语言，从而生成可执行文件。汇编语言编译器仅仅将每个指令内的词，替换为二进制代码。



# 汇编语言指令类型

## R型指令：操作寄存器

常见指令：`add`,  `sub`, `and`, `or`, `nor`, `slt`, `sll`, `srl`, `jr` 

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/Screenshot%202020-05-04%20at%2017.46.29.png?token=AJLBRHGQTP7OVFPNSPN3U526WA7MS)

- opcod: Operation Code
- rs: number of source register
- rt: number of source register
- rd: number of destination register
- sham: Shift amount (number of bits the operation is shifted)
- Func: function (opcod extension)

Sham仅用于操作偏移量的指令（例如`stl`）

## I型指令：操作常量

常见指令：`addi`, `lw`, `sw`, `lh`, `sh`, `lb`, `lbu`, `sb`, `ll`, `sc`, `lui`, `andi`, `ori`, `beq`, `bne`, `slti`, `sltiu`

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/Screenshot%202020-05-04%20at%2018.11.53.png?token=AJLBRHCWAJCSSV6TN2G4NUK6WA7Q6)

- opcod: Operation Code
- rs: number of source register
- rd: number of source or destination register
- imd: Immediate value 即时量



## J型指令：跳跃寻址

处理器直接跳跃到给定的地址，执行所在地址的指令

常见指令：`j`, `jal`

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/20200504183555.png)

- Opcod: Operation Code
- Imd: immediate value