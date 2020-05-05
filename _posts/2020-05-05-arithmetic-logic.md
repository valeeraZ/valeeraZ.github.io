---
title: MIPS算数与逻辑指令
layout: post
subtitle: Mips Arithmetic and logic instructions
date:       2020-05-05
author:     "Zhao"
header-img: "img/system.png"
tags: 
    - System
mathjax: true
---

# 常见的加减法

 ## R型指令

`add`用于实现将两个来源（source）寄存器内的值相加，并存储到目标（destination）寄存器中。然而，有时候两个寄存器内的值相加会超过32位。由于我们使用补码（Two's Complement）， 两个32位的值相加之和会达到33位。每个寄存器内的值在 $ [-2^{31} , 2^{31}-1]$区间，因此他们的和也就在$ [-2^{32} , 2^{32}-2]$区间。  

如果和超出了寄存器所存储值得区间范围，那么也就会发生**溢出（overflow）**。  

若发生位数溢出，处理器会拒绝执行`add`指令，并触发程序错误异常。  

而`addu`同为加法，但却不会判断溢出情况。  

对于减法`sub`和`subu`同理。



## I型指令

`addi rd rs imd`将即时值`imd`与寄存器`rs`中的值相加，并检查溢出情况。然而有时，rs的值是32位编码，但imd的值是16位编码，为了将这两个不同位数的值相加，需要将16位转换为32位。有两种方法：

- 如果我们认为这个imd是自然数，那么在其高位添加上16个0（也就是零位拓展）
- 如果是整数，那么将这个数的最高位（也就是第15位）复制16次添加到高位（因为是补码也就是符号位拓展）

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/20200504225046.png)

在Mips架构中，我们认为即时数是整数，因此我们先将它做符号位拓展再与寄存器内的数相加。  

进一步通俗的来说，在算数运算中（arithmetic）我们认为immediate是整数；在逻辑运算（logic）中，我们认为immediate是自然数。  

`addiu`和`addi`相同，只不过它不判断溢出。



#  逻辑运算

## R型指令

- `or rd rs rt` 或
- `and rd rs rt` 且
- `xor rd rs rt` 异或
- `nor rd rs rt` 非或

## I型指令

- `ori rd rs imd`
- `and rd rs imd`
- `xori rd rs imd`

要注意的是，此时immediate被看做是自然数，因而他执行的是零位拓展。



# 移位运算

## Logic

### 即时值sham

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/20200504231406.gif)

`sll rd rt sham`: rt >> sham => rd, shift left logical 向左移位，将最高位丢弃，每位向左移动，最低位补0。

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/20200505151747.png)

`srl rd rt sham`: rt << sham => rd 向右移位

### 寄存器值variable

`sllv rd rt rs`使用寄存器rs内的值做sham让rt向左逻辑移位  

**注意：用rs寄存器内值做sham，仅有最低五位（也就是靠右的）是有意义的. **  



## Arithmetic

![](https://raw.githubusercontent.com/valeeraZ/-image-host/master/20200505152247.png)

`sra rd rt sham`: rt >> sham => rd shift right arithmetic  向右算数移位，即保持高位数不变，因移动而缺失的位置用原数字补上。常用除法：`srl $t1, $t0, 1` 将t0的值除于2；若除于4，则移动2位。  

`srav rd rt rs`同上，用寄存器rs的值做`sra`操作。

# 比较并设置

## R型指令

`slt rd rs rt`: set if less than: `if rs < rt` then `1 => rd`, else `0 => rd`; **operands are signed** （整数）  

`sltu rd rs rt`: set if less than **unsigned**  （自然数）

## I型指令

`slti rd rs imd` set if less than immediate （sign extension注意是符号位拓展因为是算数操作）  
`slitu rd rs imd` set if less than immediate unsigned 注意当需要16位转32位时，immediate被看作整数，即使用符号位拓展，因为是算数操作。在比较时，两个比较的值都将被视作自然数！  
`lui rd imd`: load upper immediate 将imd的值载入到寄存器rd的高16位



