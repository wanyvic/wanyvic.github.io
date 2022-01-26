---
title: 零知识证明 三 zkSNARK 构建
abbrlink: '59736367'
date: 2022-01-26 12:49:42
updated: 2022-01-26 12:49:42
tags:
categories:
copyright:
---

# 开篇提示
本文是基于V神[Quadratic Arithmetic Programs: from Zero to Hero](https://vitalik.ca/general/2016/12/10/qap.html)与网友分析基础之上总结而成。

# NP问题与NPC问题

- P问题：能在多项式时间内解决的问题。
- NP问题：能在多项式时间验证答案正确与否的问题。
- NP-hard问题：任意np问题都可以在多项式时间内归约为该问题。归约的意思是为了解决问题A，先将问题A归约为另一个问题B，解决问题B同时也间接解决了问题A。
- NPC问题：既是NP问题，也是NP-hard问题。

# QAP 问题
首先，zk-SNARK 不能直接用于解决任何计算问题，我们必须先把问题转换成正确的“形式”来处理，这种形式叫做 "quadratic arithmetic problem"(QAP)，在进行 QAP 转换的同时，如果你有代码的输入就可以创建一个对应的解（有时称之为 QAP 的 witness）。之后，还需要一个相当复杂的步骤来为 witness 创建实际的“零知识证明”，此外还有一个独立的过程来验证某人发送给你的证明。

我们来选择一个简单的例子来做说明：证明你知道一个立方方程的解：$x^3 + x + 5 = 35$ （提示：解是3，解用来构造 witness ）。这个例子既可以让你理解zkSNARK背后的技术原理，又不会产生出很大的 QAP。
```python
def qeval(x):
    y = x**3
    return x + y + 5
```
我们这里所使用的特殊的程序语言支持基本算术运算 ($+,-,*,/$)，常量阶指数运算(如：可以计算 $x^7$ 但是不能计算 $x^y$ )和变量赋值，理论上可以在这个语言中做任意计算（只要计算步骤的次数是受限制的，此外，不允许循环）。注意，这个语言不支持模运算 ($\%$) 和比较运算 ($<, >, <=, >=$)，但是通过提供辅助输入可以扩展到支持这两种运算，此处我们不做展开。

# 扁平化
然后，我们把代码拍平(Flattening)：

拍平后的代码一次只能做下面的一种事情，x=y（x可以是数字或变量）。x=y op z（其中op可以是$+，-，*，/$等运算）

$
sym\_1 = x * x
$  
$
y = sym\_1 * x
$  
$
sym\_2 = y + x
$  
~$
out = sym\_2 + 5
$

在上面的过程中，我们引入了一些中间变量，但是整体跟我们的代码是等价的。
# 构建R1CS

接下来，我们需要把拍平的代码写成一个叫作R1CS（rank-1 constraint system）的约束系统。

**R1CS：** 可将其理解为一个方程组。

Prover需要向Verifier证明其知道满足该方程组所有方程式的解，证明过程可转化为Prover知道3组向量$(\vec{a},\vec{b},\vec{c})$以及1组对应于R1CS解的向量$\vec{s}$（np问题转化为np完全问题），使得$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>=0$成立。

其中$<\vec{s},\vec{a}>=\sum_{i=1}^{n}$。

拍平后的方程组：

$
sym\_1 = x * x
$  
$
y = sym\_1 * x
$  
$
sym\_2 = y + x
$  
~$
out = sym\_2 + 5
$

由方程组可知，其中涉及6个变量和4个逻辑门。

左侧组成了变量组，在此基础上加上$1$和$x$，组成了6维向量$\vec{s}=[ \text{\textasciitilde}one,x,\text{\textasciitilde}out,sym\_1,y,sym\_2 ]$

其中：
- $\text{\textasciitilde}one$：虚拟变量
- $x$：输入变量
- $\text{\textasciitilde}out$：输出变量

Prover知道满足上述方程式组的解（$x=3$），则$\vec{s}=[ \text{\textasciitilde}one,x,\text{\textasciitilde}out,sym\_1,y,sym\_2 ] = [1,3,35,9,27,30]$。

## 计算
### 第一门逻辑电路
$y=sym_1*x$，满足$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>=0$成立的$(\vec{a},\vec{b},\vec{c})$为：

$\vec{a} = [0,1,0,0,0,0]$  
$\vec{b} = [0,1,0,0,0,0]$  
$\vec{c} = [0,0,0,1,0,0]$

$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>= 3*3-9=0$


### 第二门逻辑电路
$y=sym\_1*x$，满足$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>=0$成立的$(\vec{a},\vec{b},\vec{c})$为：

$\vec{a} = [0,0,0,1,0,0]$  
$\vec{b} = [0,1,0,0,0,0]$  
$\vec{c} = [0,0,0,0,1,0]$


### 第三门逻辑电路
$sym\_2=y+x$，满足$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>=0$成立的$(\vec{a},\vec{b},\vec{c})$为：
 
因为是加法所以这里解法有些不同：  
$\vec{a} = [0,1,0,0,1,0]$  
$\vec{b} = [1,0,0,0,0,0]$  
$\vec{c} = [0,0,0,0,0,1]$

$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>= (3+27)*1-30=0$


### 第四门逻辑电路
$\text{\textasciitilde}out=sym\_2+5$，满足$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>=0$成立的$(\vec{a},\vec{b},\vec{c})$为：

$\vec{a} = [5,0,0,0,0,1]$  
$\vec{b} = [1,0,0,0,0,0]$  
$\vec{c} = [0,0,1,0,0,0]$

$<\vec{s},\vec{a}>*<\vec{s},\vec{b}>-<\vec{s},\vec{c}>= (5+30)*1-35=0$

最后得到完整的R1CS：

A:  
$[0, 1, 0, 0, 0, 0]$  
$[0, 0, 0, 1, 0, 0]$  
$[0, 1, 0, 0, 1, 0]$  
$[5, 0, 0, 0, 0, 1]$  

B:  
$[0, 1, 0, 0, 0, 0]$  
$[0, 1, 0, 0, 0, 0]$  
$[1, 0, 0, 0, 0, 0]$  
$[1, 0, 0, 0, 0, 0]$  

C:  
$[0, 0, 0, 1, 0, 0]$  
$[0, 0, 0, 0, 1, 0]$  
$[0, 0, 0, 0, 0, 1]$  
$[0, 0, 1, 0, 0, 0]$  

# R1CS to QAP
下一步是将这个 R1CS 转换成 QAP 形式，因为有6个变量，所以要对每个变量进行约束，因为有4个逻辑门所以每个多项式有4个点。它实现了完全相同的逻辑，只是使用了多项式而不是点积。
通过使用**拉格朗日插值公式**或**快速傅里叶逆变换**来将4组长度为6的三个向量转换为3组6个3次多项式。

具体作法：
从每个a向量中取出第一个值，用拉格兰奇插值来构造一个多项式(在 i 处的多项式得到 i 的第一个值)

从第一列得到$(1,0),(2,0),(3,0),(4,5)$使用这5个点用拉格朗日插值构造多项式，以此类推。

最后得到多项式

A polynomials  
$[-5.0, 9.166, -5.0, 0.833]$  
$[8.0, -11.333, 5.0, -0.666]$  
$[0.0, 0.0, 0.0, 0.0]$  
$[-6.0, 9.5, -4.0, 0.5]$  
$[4.0, -7.0, 3.5, -0.5]$  
$[-1.0, 1.833, -1.0, 0.166]$  


B polynomials  
$[3.0, -5.166, 2.5, -0.333]$  
$[-2.0, 5.166, -2.5, 0.333]$  
$[0.0, 0.0, 0.0, 0.0]$  
$[0.0, 0.0, 0.0, 0.0]$  
$[0.0, 0.0, 0.0, 0.0]$  
$[0.0, 0.0, 0.0, 0.0]$  


C polynomials  
$[0.0, 0.0, 0.0, 0.0]$  
$[0.0, 0.0, 0.0, 0.0]$  
$[-1.0, 1.833, -1.0, 0.166]$  
$[4.0, -4.333, 1.5, -0.166]$  
$[-6.0, 9.5, -4.0, 0.5]$  
$[4.0, -7.0, 3.5, -0.5]$  


系数按升序排列，所以上面的第一个多项式实际上是$0.833x^3-5x^2 + 9.166x-5$。这组多项式(加上我稍后解释的 z 多项式)构成了这个特定 QAP 实例的参数。

# 检查 QAP
我们可以将x带入多项式，例如$x=1$

A results at x=1  
$
0\\
1\\
0\\
0\\
0\\
0\\
$

B results at x=1  
$
0\\
1\\
0\\
0\\
0\\
0\\
$

C results at x=1  
$
0\\
0\\
0\\
1\\
0\\
0\\
$

这里有的，和我们上面创建的第一个逻辑门的，三个向量的集合完全一样。

与其单一检查R1CS的约束不如对多项式进行点积同时来检查所有的约束。

![](零知识证明-三-zkSNARK-构建/f2.png)

例如当$x=1$时：  
$A(1)=1*(-5)+3*8+35*0+9*(-6)+27*4+30*(-1)=43$  
$A(2)=1*9.166+3*(-11.333)+35*0+9*9.5+27*(-7)+30*1.833=-73.333$  
...

得出：  
[$A \cdot s = [43.0, -73.333, 38.5, -5.166]$](https://www.symbolab.com/solver/vector-dot-product-calculator/%5Cbegin%7Bpmatrix%7D-5%269%2B%5Cfrac%7B1%7D%7B6%7D%26-5%26%5Cfrac%7B5%7D%7B6%7D%5C%5C%20%20%20%20%20%20%20%208%26-11-%5Cfrac%7B1%7D%7B3%7D%265%26-%5Cfrac%7B2%7D%7B3%7D%5C%5C%20%20%20%20%20%20%20%200%260%260%260%5C%5C%20%20%20%20%20%20%20%20-6%269.5%26-4%26.5%5C%5C%20%20%20%20%20%20%20%204%26-7%263.5%26-.5%5C%5C%20%20%20%20%20%20%20%20-1%26%5Cfrac%7B11%7D%7B6%7D%26-1%26%5Cfrac%7B1%7D%7B6%7D%5Cend%7Bpmatrix%7D%5E%7BT%7D%5Ccdot%5Cbegin%7Bpmatrix%7D1%263%2635%269%2627%2630%5Cend%7Bpmatrix%7D%5E%7BT%7D)  
[$B \cdot s = [-3.0, 10.333, -5.0, 0.666]$](https://www.symbolab.com/solver/vector-dot-product-calculator/%5Cbegin%7Bpmatrix%7D3%26-5-%5Cfrac%7B1%7D%7B6%7D%262.5%26-%5Cfrac%7B1%7D%7B3%7D%5C%5C%20%20-2%265%2B%5Cfrac%7B1%7D%7B6%7D%26-2.5%26%5Cfrac%7B1%7D%7B3%7D%5C%5C%20%20%20%20%20%20%20%20%200%260%260%260%5C%5C%20%20%20%20%20%20%20%20%200%260%260%260%5C%5C%20%20%20%20%20%20%20%20%200%260%260%260%5C%5C%20%20%20%20%20%20%20%20%200%260%260%260%5Cend%7Bpmatrix%7D%5E%7BT%7D%5Ccdot%5Cbegin%7Bpmatrix%7D1%263%2635%269%2627%2630%5Cend%7Bpmatrix%7D%5E%7BT%7D)  
[$C \cdot s = [-41.0, 71.666, -24.5, 2.833]$](https://www.symbolab.com/solver/vector-dot-product-calculator/%5Cbegin%7Bpmatrix%7D0%260%260%260%5C%5C%200%260%260%260%5C%5C%20-1%26%5Cfrac%7B11%7D%7B6%7D%26-1%26%5Cfrac%7B1%7D%7B6%7D%5C%5C%204%26-%5Cfrac%7B26%7D%7B6%7D%261.5%26-%5Cfrac%7B1%7D%7B6%7D%5C%5C%20-6%269.5%26-4%26%5Cfrac%7B1%7D%7B2%7D%5C%5C%204%26-7%263.5%26-0.5%5Cend%7Bpmatrix%7D%5E%7BT%7D%5Ccdot%5Cbegin%7Bpmatrix%7D1%263%2635%269%2627%2630%5Cend%7Bpmatrix%7D%5E%7BT%7D)  

[$A \cdot s*B \cdot s -C\cdot s$](https://www.symbolab.com/solver/vector-dot-product-calculator/%5Cleft(-%5Cleft(5%2B%5Cfrac%7B1%7D%7B6%7D%5Cright)%20x%5E%7B3%7D%2B38.5x%5E%7B2%7D-%5Cleft(73%2B%5Cfrac%7B1%7D%7B3%7D%5Cright)x%2B43%5Cright)%5Cleft(%5Cfrac%7B2%7D%7B3%7Dx%5E%7B3%7D-5x%5E%7B2%7D%2B%5Cleft(10%2B%5Cfrac%7B1%7D%7B3%7D%5Cright)x-3%5Cright)-%5Cleft(%5Cleft(2%2B%5Cfrac%7B5%7D%7B6%7D%5Cright)x%5E%7B3%7D-24.5x%5E%7B2%7D%2B%5Cleft(71%2B%5Cfrac%7B2%7D%7B3%7D%5Cright)x-41%5Cright))$=(-5.166x^3+38.5x^2-73.333x+43)*(0.666x^3-5x^2+10.333x-3)-(2.833*x^3-24.5*x^2+71.666x-41)=-3.444x^6+51.5x^5-294.777x^4+805.833x^3-1063.777x^2+592.666x-88$

$t=[-88.0, 592.666, -1063.777, 805.833, -294.777, 51.5, -3.444]$

[$Z =(x-1)(x-2)(x-3)(x-4)= [24,-50,35,-10,1]$](https://www.symbolab.com/solver/vector-dot-product-calculator/%5Cleft(x%20-%201%5Cright)%20%5Ccdot%20%5Cleft(x%20-%202%5Cright)%20%5Ccdot%20%5Cleft(x%20-%203%5Cright)%20%5Ccdot%20%5Cleft(x%20-%204%5Cright))

[$H = t/z = [-3.666,17.055,-3.444]$](https://www.wolframalpha.com/widget/widgetPopup.jsp?p=v&id=330407de24ae45cf5e6ae7c814f1f459&title=Dividing%20Polynomials%20Calculator&theme=blue&i0=-(3%2B4/9)*x%5E6%2B51.5*x%5E5-(294%2B7/9)*x%5E4%2B(805%2B5/6)*x%5E3-(1063%2B7/9)*x%5E2%2B(592%2B2/3)*x-88&i1=x%5E4-10x%5E3%2B35x%5E2-50x%2B24&podSelect=&includepodid=Input&includepodid=QuotientAndRemainder&podstate=QuotientAndRemainder__Step-by-step%20solution&showAssumptions=1&showWarnings=1)

精确值：$H = t/z = -\frac{31}{9}x^2+\frac{307}{18}x-\frac{66}{18}$

即：t可被z整除。

# 参考资料
> https://james-christopher-ray.medium.com/here-are-the-calculations-for-the-product-of-a-b-and-c-with-s-ae6a3f94647a  
> https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649