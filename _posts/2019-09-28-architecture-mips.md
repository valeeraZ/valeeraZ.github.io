---
title: MIPS汇编语言入门
layout: post
subtitle: MIPS：无互锁流水级的微处理器(Microprocessor without Interlocked Piped Stages) 
date:       2019-09-28
author:     "Zhao"
header-img: "img/system.png"
tags: 
    - System
---
摘自维基百科：  
MIPS架构（英语：MIPS architecture，为Microprocessor without Interlocked Pipeline Stages的缩写，亦为Millions of Instructions Per Second的头字语），是一种采取精简指令集（RISC）的处理器架构，1981年出现，由MIPS科技公司开发并授权，广泛被使用在许多电子产品、网络设备、个人娱乐设备与商业设备上。最早的MIPS架构是32位，最新的版本已经变成64位。  
在1981年，斯坦福大学教授约翰·轩尼诗领导他的团队，实现出第一个MIPS架构的处理器。他们原始的概令是透过指令管线化来增加CPU运算的速度。
1984年，约翰·轩尼诗教授离开斯坦福大学，创立MIPS科技公司。于1985年，设计出R2000芯片，1988年，将其改进为R3000芯片。

**版权声明:**  
本文部分资料来源于CSDN博主thoupin的博文[MIPS编程入门](https://www.cnblogs.com/thoupin/p/4018455.html)。转载请附上此地址及本文地址。

<!-- TOC -->

- [机器周期](#机器周期)
- [汇编语言](#汇编语言)
- [MIPS](#mips)
    - [寄存器种类](#寄存器种类)
    - [程序结构](#程序结构)
        - [代码](#代码)
        - [数据声明](#数据声明)
        - [常用指令](#常用指令)
            - [加载/保存(意译为 读取/写入)](#加载保存意译为-读取写入)
            - [立即与间接寻址](#立即与间接寻址)
            - [算术指令](#算术指令)
            - [控制流](#控制流)
            - [系统指令](#系统指令)

<!-- /TOC -->

# 机器周期

大多数计算机处理器都会不断地重复三个基本步骤。每个机器周期内会执行一条机器指令。一个现代的计算机处理器每秒钟运行数百万次机器周期。  
一条机器指令是由一串对应着处理器基本操作的二进制码组成的，在不同的处理器架构中，机器周期的组成也不相同，但他们的基本行为都包含下面三个主要步骤： 

- 从内存中读取指令：指令存放在内存中，PC *(Program Counter)* 存放了指令在内存中的地址

- PC=PC+4：让PC指向下一条指令所在的地址

- 执行所得到的指令

在一台32位的处理器上，内存地址为32位宽，因此PC也拥有长度为32位的地址

# 汇编语言

机器指令是由0,1的二进制码构成的因而人类无法阅读。相对应地，汇编语言允许我们使用相应的代码来编写指令。下面是对应的机器码和汇编语言：

machine instruction  
`0000 0001 0010 1011 1000 0000 0010 0000`

assembly language statement  
```mipsasm
add $t0,$t1,$t2
```

这条指令的意思是：寄存器$t0,$t1,$t2间
```
$t0 = $t1 + $t2
```

![conceputual view.jpg](https://i.loli.net/2019/09/29/pTnSiez2c94Oakb.jpg "Conceptual view")

# MIPS

## 寄存器种类

- MIPS下一共有32个通用寄存器
- 在汇编中，寄存器标志由$符开头
- 寄存器表示可以有两种方式
  - 直接使用该寄存器对应的编号，例如：从$0到$31
  - 使用对应的寄存器名称，例如：$t1, $sp
- 对于乘法和除法分别有对应的两个寄存器$lo, $hi
  - 对于以上二者，不存在直接寻址；必须要通过mfhi("move from hi")以及mflo("move from lo")分别来进行访问对应的内容
- 栈的走向是从高地址到低地址

REGISTER|NAME|USAGE
|:-:|:-:|:-:|
$0|$zero|常量0(constant value 0)
$1|$at|保留给汇编器(Reserved for assembler)
$2-$3|$v0-$v1|函数调用返回值(values for results and expression evaluation)
$4-$7|$a0-$a3|函数调用参数(arguments)
$8-$15|$t0-$t7|暂时的(或随便用的)
$16-$23|$s0-$s7|保存的(或如果用，需要SAVE/RESTORE的)(saved)
$24-$25|$t8-$t9|暂时的(或随便用的)
$28|$gp|全局指针(Global Pointer)
$29|$sp|堆栈指针(Stack Pointer)
$30|$fp|帧指针(Frame Pointer)
$31|$ra|返回地址(return address)

说明

1. $0:即$zero,该寄存器总是返回零，为0这个有用常数提供了一个简洁的编码形式。  
```mipsasm
move $t0,$t1
#实际为  
add $t0,$0,$t1
```
使用伪指令可以简化任务，汇编程序提供了比硬件更丰富的指令集。
2. $1:即$at，该寄存器为汇编保留，由于I型指令的立即数字段只有16位，在加载大常数时，编译器或汇编程序需要把大常数拆开，然后重新组合到寄存器里。比如加载一个32位立即数需要 `lui`（装入高位立即数）和`addi`两条指令。像MIPS程序拆散和重装大常数由汇编程序来完成，汇编程序必需一个临时寄存器来重组大常数，这也是为汇编保留$at的原因之一。
3. $2..$3:($v0-$v1)用于子程序的非浮点结果或返回值，对于子程序如何传递参数及如何返回，MIPS范围有一套约定，堆栈中少数几个位置处的内容装入CPU寄存器，其相应内存位置保留未做定义，当这两个寄存器不够存放返回值时，编译器通过内存来完成。
4. $4..$7:($a0-$a3)用来传递前四个参数给子程序，不够的用堆栈。a0-a3和v0-v1以及ra一起来支持子程序／过程调用，分别用以传递参数，返回结果和存放返回地址。当需要使用更多的寄存器时，就需要堆栈（stack)了,MIPS编译器总是为参数在堆栈中留有空间以防有参数需要存储。
5. $8..$15:($t0-$t7)临时寄存器，子程序可以使用它们而不用保留。
6. $16..$23:($s0-$s7)保存寄存器，在过程调用过程中需要保留（被调用者保存和恢复，还包括$fp和$ra），MIPS提供了临时寄存器和保存寄存器，这样就减少了寄存器溢出（spilling,即将不常用的变量放到存储器的过程),编译器在编译一个叶（leaf)过程（不调用其它过程的过程）的时候，总是在临时寄存器分配完了才使用需要保存的寄存器。
7. $24..$25:($t8-$t9)同($t0-$t7)
$26..$27:($k0,$k1)为操作系统／异常处理保留，至少要预留一个。 异常（或中断）是一种不需要在程序中显示调用的过程。MIPS有个叫异常程序计数器（exception program counter,EPC)的寄存器，属于CP0寄存器，用于保存造成异常的那条指令的地址。查看控制寄存器的唯一方法是把它复制到通用寄存器里，指令`mfc0`(move from system control)可以将EPC中的地址复制到某个通用寄存器中，通过跳转语句（`jr`)，程序可以返回到造成异常的那条指令处继续执行。MIPS程序员都必须保留两个寄存器$k0和$k1，供操作系统使用。  
发生异常时，这两个寄存器的值不会被恢复，编译器也不使用k0和k1,异常处理函数可以将返回地址放到这两个中的任何一个，然后使用jr跳转到造成异常的指令处继续执行。
8. $28:($gp)为了简化静态数据的访问，MIPS软件保留了一个寄存器：全局指针gp(global pointer,$gp)，全局指针指向静态数据区中的运行时决定的地址，在存取位于gp值上下32KB范围内的数据时，只需要一条以gp为基指针的指令即可。在编译时，数据须在以gp为基指针的64KB范围内。
9. $29:($sp)MIPS硬件并不直接支持堆栈，你可以把它用于别的目的，但为了使用别人的程序或让别人使用你的程序，还是要遵守这个约定的，但这和硬件没有关系。
10. $30:($fp)GNU MIPS C编译器使用了帧指针(frame pointer),而SGI的C编译器没有使用，而把这个寄存器当作保存寄存器使用（$s8),这节省了调用和返回开销，但增加了代码生成的复杂性。
11. $31:($ra)存放返回地址，MIPS有个`jal`(jump-and-link,跳转并 链接)指令，在跳转到某个地址时，把下一条指令的地址放到$ra中。用于支持子程序，例如调用程序把参数放到$a0~$a3,然后`jal X`跳到X过程，被调过程完成后把结果放到$v0,$v1,然后使用`jr $ra`返回。

## 程序结构

### 代码

- 代码段以 .text为开始标志
- 包含了各项指令操作
- 程序入口为main：标志
- 程序结束标志
- 注释以#开始

### 数据声明

- 数据段以 .data为开始标志
- 声明变量后，即在主存中分配空间。

格式：  
`name: storage_type value(s)`  

- name:变量名
- storage_type:数据类型，可以为.word, .ascii, .data, .byte等等
  - 对于.space 需要指明需要多大空间(bytes)
  - .ascii 与 .asciiz唯一区别就是 后者会在字符串最后自动加上一个终止符
- value:初始值

例子：

```mipsasm
.data
var1: .word 3 # 声明一个 word 类型的变量 var1, 同时给其赋值为 3
array1: .byte 'a','b' # 声明一个存储2个字符的数组 array1，并赋值 'a', 'b'
array2: .space 40 # 为变量 array2 分配 40字节（bytes)未使用的连续空间，最好事先注释数据类型
```

### 常用指令

#### 加载/保存(意译为 读取/写入)  

如果要访问内存，只能用 load 或者 store 指令，其他的只能都一律是寄存器操作  

```mipsasm
#load word
lw register_destination, RAM_source
#copy word (4 bytes) at source RAM location to destination register.
#从内存中 复制 RAM_source 的内容到 对应的寄存器中（lw中的'w'意为'word',即该数据大小为4个字节）

lb register_destination, RAM_source
#copy byte at source RAM location to low-order byte of destination register,
# and sign-e.g.tend to higher-order bytes
#从内存RAM location中复制低端位的byte至目标寄存器；同上， lb 意为 load byte

#store word
sw register_source, RAM_destination
#store word in source register into RAM destination

sb register_source, RAM_destination
#store byte (low-order) in source register into RAM destination
#将指定寄存器中的数据 写入 到指定的内存中
```

例子：

```mipsasm
.data
var1: .word 23
# declare storage for var1; initial value is 23
# 先声明一个 word 型的变量 var1 = 3;

.text
__start:
lw $t0, var1
# load contents of RAM location into register $t0: $t0 = var1
# 令寄存器 $t0 = var1 = 3;

li $t1, 5
# $t1 = 5   ("load immediate")
# 令寄存器 $t1 = 5;

sw $t1, var1
# store contents of register $t1 into RAM: var1 = $t1
# 将var1的值修改为$t1中的值： var1 = $t1 = 5;
```


#### 立即与间接寻址  
  
  
```mipsasm
#直接寻址:  
la $t0, var1
#copy RAM address of var1 (presumably a label defined in the program) into register $t0
#将var1在内存中的地址放入$t0，可以理解为指针

#间接寻址
lw $t2, ($t0)
#load word at RAM address contained in $t0 into $t2
#将 内存中地址指向为$t0的词 保存在 寄存器$t2

sw $t2, ($t0)
#store word in register $t2 into RAM at address contained in $t0
#将 寄存器$t2中的词 保存在 内存地址指向为$t0的内存中


#+偏移量
lw $t2, 4($t0)
#load word at RAM address ($t0+4) into register $t2
"4" gives offset from address in register $t0

sw $t2, -12($t0)
#store word in register $t2 into RAM at address ($t0 - 12)
#negative offsets are fine
#括号前的数字为地址偏移量，每四字节为一空间，+4即为下一个空间

#不必多说，要用到偏移量的寻址，基本上使用最多的场景无非两种：数组，栈。
#Note: based addressing is especially useful for:
#arrays; access elements as offset from base address
#stacks; easy to access elements at offset from stack pointer or frame pointer
```

#### 算术指令

- 最多3个操作数
- 再说一遍，在这里，操作数只能是寄存器，绝对不允许出现地址
- 所有指令统一是32位 = 4 * 8 bit = 4bytes = 1 word  

```mipsasm
add $t0,$t1,$t2 # $t0 = $t1 + $t2; add as signed (2's complement) integers

sub $t2,$t3,$t4     #$t2 = $t3 Ð $t4
addi $t2,$t3, 5     #$t2 = $t3 + 5;   "add immediate" (no sub immediate)
addu $t1,$t6,$t7    #$t1 = $t6 + $t7;   add as unsigned integers
subu $t1,$t6,$t7    #$t1 = $t6 + $t7;   subtract as unsigned integers

mult $t3,$t4        
#multiply 32-bit quantities in $t3 and $t4, and store                       64-bit result in special registers Lo and Hi:  (Hi,Lo) = $t3 * $t4　　　　　　　　　 
#运算结果存储在hi,lo（hi高位数据， lo地位数据）

div $t5,$t6		    
#  Lo = $t5 / $t6   (integer quotient)				    
#  Hi = $t5 mod $t6   (remainder)
#  商数存放在 lo, 余数存放在 hi

mfhi $t0		    
#move quantity in special register Hi to $t0:   $t0 = Hi
#不能直接获取 hi 或 lo中的值， 需要mfhi, mflo指令传值给寄存器

mflo $t1		
#move quantity in special register Lo to $t1:   $t1 = Lo
#used to get at result of product or quotient

move $t2,$t3	#$t2 = $t3复制粘贴
```

#### 控制流

- 分支if else  
comparison for conditional branches is built into instruction  
```mipsasm
b	target		    #  unconditional branch to program label target
beq	$t0,$t1,target	#  branch to target if  $t0 = $t1
blt	$t0,$t1,target	#  branch to target if  $t0 < $t1
ble	$t0,$t1,target	#  branch to target if  $t0 <= $t1
bgt	$t0,$t1,target	#  branch to target if  $t0 > $t1
bge	$t0,$t1,target	#  branch to target if  $t0 >= $t1
bne	$t0,$t1,target	#  branch to target if  $t0 <> $t1
bgez $t0,target     #  branch to target if $t0 >= 0
bgtz $t0,target     #  branch to target if $t0 > 0
blez $t0,target     #  branch to target if $t0 <= 0
bltz $t0,target     #  branch to target if $t0 < 0  
```


- 跳转（while, for, goto系列）
```mipsasm
j	target	　　　　 #  unconditional jump to program label target
                    #  不考虑任何条件，直接跳转至label指向的子程序/过程
jr	$t3		        #  jump to address contained in $t3 ("jump register")
　　　　　　　　　　　#  类似相对寻址，跳到该寄存器给出的地址处
```

其他指令详询[常用mips指令](https://e-mailky.github.io/2017-09-07-mips_instruction)


- 子程序调用 Subroutine Calls

```mipsasm
#subroutine call: "jump and link" instruction
jal	sub_label	#  "jump and link"
#copy program counter (return address) to register $ra (return address register)
#将当前的程序计数器保存到 $ra 中
#jump to program statement at sub_label
#并跳转至子程序sub_label

#subroutine return: "jump register" instruction
jr	$ra	
#"jump to register"
#jump to return address in $ra (stored by jal instruction)
#通过上面保存在 $ra 中的计数器返回调用前
```
Note: return address stored in register $ra; if subroutine will call other subroutines, or is recursive, return address should be copied from $ra onto stack to preserve it, since jal always places return address in this register and hence will overwrite previous value
如果说调用的子程序中有调用了其他子程序，如此往复， 则返回地址的标记就用 栈（stack） 来存储, 毕竟 $ra 只有一个

#### 系统指令

- 打印寄存器中的值
必须将值放在寄存器$4中，通过对$2赋值打印不同类型的数据  
例：
```mipsasm
li  $4, 12345678    #将整数值12345678放在寄存器$4中
li  $2, 1           #对$2赋值1
syscall             #将值打印在终端
```
对于不同的数据类型，赋给$2的值也不同

Value for $2 to print|type of data
:-:|:-:
1|integer
2|float
3|double
4|string
11|character

- 从终端读取值，并存放在寄存器$2中
```mipsasm
li  $2, 5           #对$2赋值5
syscall             #读取终端的值，并放入$2中
```
对于不同的数据类型，赋给$2的值也不同

Value for $2 to read|type of data
:-:|:-:
5|integer
6|float
7|double
8|string
12|character

注意：对于读取字符串，需规定读取的字符串最大长度
```mipsasm
read:   .space 256  #声明一个256长度的内存中buffer
        la $4,read  #将$4指向read地址
        li $5,255   #读取的字符串最大长度为255
        li $2,$8    #读取
        syscall     
```

- 程序退出
```mipsasm
li  $2,10
syscall
```

----
**参考资料:**  
[中康涅狄格州州立大学课程](https://chortle.ccsu.edu/AssemblyTutorial/index.html)  
[巴黎六大-索邦大学计算机软件及硬件组成原理](https://www-soc.lip6.fr/trac/sesi-almo/)
