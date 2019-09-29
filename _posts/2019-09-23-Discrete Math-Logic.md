---
title: 离散数学-逻辑与证明
layout: post
subtitle: 离散数学是计算机专业很重要的基础课程，是后续数据结构，算法的基础。
date:       2019-09-23
author:     "Zhao"
header-img: "img/algo.jpg"
tags: 
    - Math
jax:    "1"
---
# 命题逻辑

## 何为命题  

定义：命题是一个能判断真假的陈述句。  
既然能判断真假，就意味着一个命题或者为真，或者为假，不能既真又假。
例如：“地球是圆的”，“太阳系有八大行星”，“时间是静止的”，都是命题，其中前两个命题为真，后一个为假。  
我们一般使用命题变元来表示命题，如p，q，r，s，t…  
如果命题为真命题，则称其真值为真，记作T（即True）。反之，假记作F（即False）。

## 逻辑运算符

写法、读法及真值如下表：  

逻辑运算符|符号|意义|读法|真值
:-:|:-:|:-:|:-:|:-:
合取|p∧q|p 和 q 的合取|p 且 q|p、q 同为真时为真，有一个为假则为假
析取|p∨q|p 和 q 的析取|q 或 q|p、q 同为假时为假，有一个为真则为真
否定|¬p|p 的否定|非 p|¬q 与 q 真值相反
异或|p⊕q|p 和 q 的异或|p 异或 q|p、q 真值相同为假，相异为真

## 条件命题

逻辑连接词|命题|含义（表达）|真值  
|:-:|:-:|:-:|:-:|
→|p→q|p 蕴含 q、如果 p，则 q 、等|p 真 q 假时为假，其余情况为真
↔|p↔q|p 双向蕴含 q、p 当且仅当 q、等|p、q 真值相同为真，相异为假

命题语言是一种**人工语言**，并不以自然语言的用法为基础。
数学中命题语言较自然语言范围更加宽泛。  
数学中的条件语句是一个**数学概念**，并不依赖于假设与结论之间的因果关系。  

p→q 表达的是 p 能推出 q 的含义，即是：p是q的充分条件。这是一个命题，真假待判断。  
同理，p↔q 表示 p 是 q 的充要条件。

由条件命题（p→q）可以构造出一些新的条件命题，如它的逆命题（q→p），否命题（¬p→¬q），逆否命题（¬q→¬p）。

## 复合命题

由逻辑连接词 ¬, ∧, ∨, → ,↔ 连接的命题称为复合命题。符合命题的真值由复合命题中各命题变元所确定。

一个真值永远为真的复合命题称为**永真式或重言式**，恒为假则称为**矛盾式**。

逻辑连接词具有优先级 ¬, ∧, ∨, → ,↔ 从左到右依次降低。

## 命题等价

在所有情况下都有相同真值（真值表完全相同）的两个命题称为是逻辑等价的。

学习了双条件语句后，也可以如此定义：
如果 p↔q 为永真式，则 p 和 q 称为逻辑等价的。记作 p≡q 或者 p⟺q 。

注意：其中 ≡ 并不是逻辑连接词，p≡q 不再是一个复合命题，而是表示 “p↔q 为永真式”。

以下为很多很重要的逻辑等价式，在化简证明中可以直接使用，其正确性都可以通过真值表来证明。并未完全列出。

### 逻辑运算符连接的逻辑等价式

德摩根律  
¬(p∧q)≡¬q∨¬q  
¬(p∨q)≡¬q∧¬q  
德摩根律可以拓展到多个命题变元。

分配律  
p∨(q∧r)≡(p∨q)∧(p∨r)  
p∧(q∨r)≡(p∧q)∨(p∧r)  

吸收律  
p∨(p∧q)≡p  
p∧(p∨q)≡p  

### 条件命题逻辑等价式

p→q≡¬p∨q  
p→q≡¬q→¬q（条件命题等价于其逆否命题）

### 双条件命题逻辑等价式

p↔q≡(p→q)∧(q→p)

# 量词逻辑

## 谓词

谓词：句子中代表主语属性的那一部分。  
例如：可以用命题函数P(x)表示命题“x>3”，其中x是变量，P则表示谓词“大于3”。  
一般地，涉及n个变量的命题可以表示为 P(x1,x2,…,xn)。P称为命题函数，或n元谓词。

## 量词一览

量化|全称量化|存在量化
:-:|:-:|:-:
量词|全称量词 ∀|存在量词 ∃
表示|∀xP(x)|∃xP(x)
读作|对所有x，P(x)|存在x，P(x)
否定|∃x¬P(x)|∀x¬P(x)

还有其他的量化，比如存在一个（恰有一个，唯一性量词∃!xP(x)），存在几个等，这里主要考虑全称量化和存在量化。对应的命题称为全称命题和存在命题。

其中x的考虑范围称为**论域**。

## 优先级

量词的优先级高于前面的所有逻辑运算符。

## 涉及量词的逻辑等价式

∀x(P(x)∧Q(x))≡∀xP(x)∧∀xQ(x)  
∃x(P(x)∨Q(x))≡∃xP(x)∨∃xQ(x)  
这很容易理解，反之，全称量词对析取不可分配，存在量词对合取也是不可分配的。

量词的德摩根律：  
¬∀xP(x)≡∃x¬P(x)  
¬∃xP(x)≡∀x¬P(x)  
为什么这也叫德摩根律呢？当变量x的论域是离散数据的集合时，上述命题便可化为多个命题变元的德摩根律。

# 嵌套量词

## 量词的嵌套

量词可以嵌套。  
例如语句：“两个正整数之和一定是正数”便可以表示为 ∀x>0∀y>0(x+y>0)，或者∀x∀y((x>0)∧(y>0)→(x+y>0))，其中论域为整数。
或者∀x∀y(x+y>0)，其中论域为正整数。

可以看到，论域不同，表示也可以不同。

多重量化中，量词的顺序很重要，不同的顺序将表示不同的含义，除非全部是全称量词或者存在量词。

## 例子

让我们回顾极限 $\lim_{x\to a}f(x) = L$
定义是：对任意实数 ϵ>0，存在一个实数 δ>0，使得对任意的 x ，只要有 0<|x−a|<δ，就有0<|f(x)−L|<ϵ.  
则使用嵌套量词可表示为：$\forall \epsilon >0 \ \exists \delta >0 \forall x(0<|x-a|<\delta \to 0<|f(x)-L|<\epsilon)$
其中 ϵ，δ，x 的论域为实数集合。

## 嵌套量词的否定

嵌套量词的否定可以通过连续应用量词的德摩根律得到。  
比如对于“ $\lim_{x→a}f(x)$ 不存在”这个命题：对于全体实数L，$\lim_{x→a}f(x)≠L$  
使用量词表示为即为：$\forall L \ \exists \epsilon >0 \ \forall \delta >0 \exists x(0<|x-a|<\delta \land |f(x)-L| \geq \epsilon)$
其中 L，ϵ， δ，x 的论域为实数集合。

## 自然语言与量词逻辑的翻译

我们可以将涉及到量词的自然语言翻译为量词逻辑。  
例题（Discrete Mathematics and Its Application,Chapter1,section 1.5,Practice No.9）：  
令L(x,y)表示x爱y，其中x和y论域为所有人的集合。
则下列语句的可如此翻译：

- 每个人都爱Jerry——∀xL(x,Jerry)
- 每个人都爱某个人——∀x∃yL(x,y)
- 有个每个都不爱的人——∃y∀x¬L(x,y)
- 恰有一个每个人都爱他的人——∃!x∀yL(y,x) 或 ∃x(∀yL(y,x)∧∀z((∀wL(w,z))→z=x))
- 有人除了自己以外谁都不爱——∃x∀y(L(x,y)↔x=y) 或 ∃x(L(x,x)∧∀y(y≠x→¬L(x,y)))

# 推理规则

## 有效论证

所谓论证：一连串的命题，并以结论最后的命题。  
所谓前提：结论之前的所有命题。  
所谓结论：最后一个命题。  
所谓有效性：结论必须由前提的真实性得出。  
所谓谬误：错误的推理，将导致无效论证。  
所谓有效论证：“前提蕴含结论”这个条件命题为**永真式**。  

当命题 $p_1\land p_2\land\dots\land p_n\to q$为永真式时，论证有效，其中p1…pn为前提，q为结论。  
其中**如果前提(p1∧p2∧⋯∧pn)为真时，结论(q)为真，则该论证是有效的**。  
这时我们应该思考为什么，回顾条件命题 p→q 的真值表：

p|q|p→q
|:-:|:-:|:-:
T|T|T
T|F|F
F|T|T
F|F|T

可以看到当前提为假时，条件命题是为真的，所以证明一个论证是有效的，只需要证明当前提为真时，结论一定为真即可。  
利用上述真值表，当要证明条件命题为真时，我们也可以通过证明前提为假来实现（这也是一种证明思路，后面会叙述）。

## 命题逻辑的推理规则

我们可以通过先建立一些简单的有效论证形式来作为推理规则，并以此为基础来构造出更复杂的有效论证形式。  
下面是一些最基础的推理规则（给出的是该有效论证形式基于的永真式）：  

推理规则|永真式
|:-:|:-:|
假言推理|((p→q)∧p)→q
取拒式|(¬q∧(p→q))→¬p
假言三段论|((p→q)∧(q→r))→(p→r)
析取三段论|((p∨q)∧¬p)→q
附加律|p→(p∨q)
化简律|(p∧q)→p
合取律|((p)∧(q))→p
消解律|((p∨q)∧(¬p∨r))→(q∨r)

都是非常容易理解和证明的，现证明一下假言推理。  
((p→q)∧p)→q  
≡¬((¬p∨q)∧p)∨q  
≡¬(p∧q)∨q  
≡¬p∨¬q∨q  
≡T  

## 谬误

((p→q)∧q)→p不是永真式，如果不小心使用了进行推理，则会产生**肯定结论的谬误**。

(¬p∧(p→q))→¬q也不是永真式，使用其进行推理会产生**否定假设的谬误**。

## 量化命题的推理规则

量化命题的推理规则有全称实例，全称引入，存在实例和存在引入。非常简单，不予赘述。

当量化命题与其前面的推理规组合使用时，有：  
全称假言推理：∀x(P(x)→Q(x))∧P(a)→Q(a)  
全称取拒式：∀x(P(x)→Q(x))∧¬Q(a)→¬P(a)  

# 证明

## 一些专业术语

定理：一个能够被证明为真的语句。  
证明：展示一个定理为真的论证。  
引理：帮助证明其他结论的定理。  
推论：从一个已被证明的定理可以直接建立起来的定理。  
猜想：被提出认为是真的定理。  

## 证明定理的方法

在现实学习中，我们已经直接或者间接的用到了这些方法和前面的推理规则，只是并没有把他们显式地提出来罢了。

这里为证明命题p→q，有下面的方法。

### 直接证明法

通过证明如果 p 为真，那么 q 也为真来证明定理的方法。  
相对应地其他方法称为间接证明法。

### 反证法

利用 p→q≡¬q→¬p，通过证明 ¬q→¬p 为真来证明 p→q 为真的方法。

即是，先假设前提 q 不成立（¬q为真），推出 p 不成立（¬p为真），与条件矛盾，进而推出 q 为真。

### 空证明

根据 p→q 的真值表，我们知道 p 为假时，p→q 为真。

所以通过证明前提为假来证明条件命题为真的方法称为空证明。
这时不需要考虑结论的真假。

### 平凡证明

根据 p→q 的真值表，我们还知道 q 为真时，p→q 为真。
所以通过证明结论为真来证明条件命题为真的方法称为平凡证明。
这里不需要考虑前提的真假（因为任何情况下结论均为真）。

### 归谬证明法

找到一个矛盾式 q 使得 ¬p→q 为真，则 ¬p 为假，即 p为真。

### 等价证明法

利用等价式：(p↔q)≡(p→q)∧(q→p)
通过证明 p→q为真且 q→p为真来证明双条件命题 p↔q为真的犯法称为等价证明法。

也即是我们常用的，证明 p 当且仅当 q 或者 p 等价于 q 时分别证明充分性和必要性。

### 反例证明法

如果需要证明 ∀xP(x) 为假，则只需找出一个反例即可。

同理，对于 ∃xP(x) 只需要找到一个x使得P(x)成立即可。

### 构造证明的方法与策略

有一些比较特殊的证明可以用特殊的方法：  
如存在性证明可以有构造证明法和非构造性证明法。  
唯一性证明需要同时证明唯一性和存在性。  

**参考资料：《离散数学及其应用》：Kenneth H.Rosen著**