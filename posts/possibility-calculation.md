---
date: 2024-04-03
update: 2024-06-19
title: 游戏王MD中的概率问题
tags:
  - math
description: 怎么玩MD老是后手...我要成为游戏王（划掉）概率论糕手！
---

# {{ $frontmatter.title }}

## 前言

:::details 游戏王的先后手差距
游戏王/YGO是一个先后手差距非常大的卡牌游戏。

虽然有各种各样的“手坑”让后手玩家可以在先手玩家的第一回合进行干扰，然而作用有限：

- 先手玩家可以无效它们
- 如今的卡组即使受到少量干扰也常常能做出妥协场面形成压制
- 先手玩家也能使用手坑来干扰后手玩家（点名批评增殖的G）

因此即使先手少抽1张牌并且没有战斗阶段，先手仍然有着巨大的优势，以致于硬币的正反很大程度上左右最终的胜负。
:::

![心肺骤停](https://s2.loli.net/2024/04/28/dHTUECXtWiOeVc6.png)

在一个抛硬币决定先后的游戏里喜提n连硬币反面概率有多大呢？~~是谁11连后手跑来算概率~~

算是（久违地）引起了我整点数学的兴趣。由此顺便联想到了不少相关的概率问题（某卡组升段/掉段的概率、bo3连胜制的胜率、...），零零碎碎算了不少内容最终成了这篇文章。

> 改成猜拳决定先后的话概率上是没什么区别的，还是会有倒霉蛋喜提10+连后手。

:::info 游戏的胜负也不过是复杂的抛硬币
抛硬币是个相当通用的例子，各种概率恒定的独立事件都可以抽象为硬币来处理。比如已知卡组胜率连胜$n$场的概率之类的（当然正经算的话还得考虑先后手因素）。所以本文的计算也没有直接使用$p=\dfrac{1}{2}$的均匀硬币模型，这样还有个好处：当你怀疑“硬币”的时候可以拿来做**贝叶斯估计**XD
:::

## TL;DR

我把这篇文章的主要计算结果都放在这里，如果你对冗长的计算没什么兴趣也许可以不看之后的计算。~~有些结果就已经很冗长了~~

### 抛m次硬币连续n次正面问题

假设单次硬币正面概率为$p$，那么为了出现连续$n$次硬币，期望的抛硬币次数是

$$
E(m)=\dfrac{1-p^n}{p^n(1-p)}
$$

抛了$m$次硬币，有连续$n$次正面的概率是

$$
P(Z_m \geq n) = \sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j-1}(p-1)^{j-1}p^{nj}\left[p+\dfrac{(1+m-nj)(1-p)}{j}\right]
$$

当$m\gg n$且$n$充分大时，

$$
P(Z_m \geq n) \sim 1-\mathrm{e}^{(m+1)(p-1)p^n}
$$

### 实卡无穷对局连胜K局问题

AB双方无限对局，由A首先先攻，单局胜者下一局后攻。当某一方首先获得单局$K$连胜时视为游戏胜利。第一局（主卡局），卡组A对卡组B的先攻胜率为$p_0$；在备牌局，卡组A的先攻胜率为$p_1$，后攻胜率为$p_2$。

分出胜负所需局数的期望$E$由以下方程组给出

$$
\begin{dcases}
  E_1=\left(E_{-1}+\dfrac{1}{1-p_2}\right)\left(1-p_2^{K-1}\right) \\
  E_{-1}=\left(E_{1}+\dfrac{1}{p_1}\right)\left(1-(1-p_1)^{K-1}\right)\\
  E=(1-p_0)E_{-1}+p_0E_1+1
\end{dcases}
$$

A获得最终游戏胜利的概率是

$$
p^+_{K}=\dfrac{p_2^{K-1}\left[1-(1-p_0)(1-p_1)^{K-1}\right]}{(1-p_1)^{K-1}+p_2^{K-1}-(1-p_1)^{K-1}p_2^{K-1}}
$$

特别地，当$K=2$时，

$$
\begin{gather*}
  E=\dfrac{2+p_0+p_1p_2-p_0(p_1+p_2)}{1-p_1+p_1p_2}\\
  p^+=\dfrac{p_2(p_0+p_1-p_0p_1)}{1-p_1+p_1p_2}
\end{gather*}
$$

如果双方使用完全相同的卡组（含side），那么$p_1+p_2=1$；如果双方在MD打，没有side，那么$p_0=p_1$。

### MD天梯升降段问题

设某卡组单局游戏胜率为$p=\dfrac{p_{\text{先}}+p_{\text{后}}}{2}$，假设刚升上某一小段，问升段/降段发生的期望局数，以及升段的概率。

#### 钻石段/白金段

累计4胜升段，0胜场3连败掉段。

$$
\begin{gather*}
E=\dfrac{2p^5-4p^4-p^3+10p^2-6p+3}{p^6-5p^5+11p^4-13p^3+11p^2-5p+1}\\
p^+=\dfrac{p^4(p^2-3p+3)}{p^6 - 5 p^5 + 11 p^4 - 13 p^3 + 11 p^2 - 5 p + 1}
\end{gather*}
$$

#### 大师段

累计5胜升段，0胜场3连败掉段。

$$
\begin{gather*}
E=\dfrac{3p^6 - 9p^5 + 12p^4 - 11p^3 + 16p^2 - 9p + 3}{2p^6 - 10p^5 + 22p^4 - 24p^3 + 16p^2 - 6p + 1}\\
p^+=\dfrac{p^5(p^2 - 3p + 3)}{2p^6 - 10p^5 + 22p^4 - 24p^3 + 16p^2 - 6p + 1}
\end{gather*}
$$

$p=0.5$时，钻石段升段概率为$63.6\%$，大师段升段概率为$58.3\%$。单局胜率低于$50\%$的卡组升段的概率可以大于$50\%$。钻石段升降段期望局数约为13，大师段升降段期望局数约为10。

#### 一般情况

一般地，如果某小段需要累计$N$胜升段，0胜场$M$连败掉段，卡组单局胜率为$p$。那么，能成功升段的概率为：

$$
p^+=\dfrac{-f(-M)}{f(N)-f(-M)}
$$

其中，

$$
f(n)=\begin{dcases}
  \dfrac{p}{2p-1}\left[1-\left(\dfrac{1-p}{p}\right)^n\right] &,n\geq 0\\
  1-(1-p)^n &,n<0
\end{dcases}
$$

而升降段发生所需的期望局数$E$由以下线性方程组给出：

$$
\begin{dcases}
  E_1=\dfrac{N-1}{2p-1}+\left(E_0-\dfrac{N}{2p-1}\right)\dfrac{p^N(1-p)-p(1-p)^N}{p^{N+1}-p(1-p)^N}\\
  E=\left(E_1+\dfrac{1}{p}\right)\left[1-(1-p)^M\right]
\end{dcases}
$$

## 抛多少次硬币会出现连续n次正面？

首先我们想知道大概抛多少次硬币会出现连续$n$次正面，换句话说，想要知道出现连续$n$次正面所需要的投掷次数的**期望**。

抛一枚硬币，记正面的概率为$p$. 记恰好$m$次投掷硬币获得$n$次连续正面的概率为$P_{m, n}$

显然有，$P_{n, n} = p^{n}$ 以及 $P_{m, n} = 0\ (m < n)$

对于$m>n$，即前$n$次没能投出全正面的情形。我们考虑首次出现反面时的投掷次数$i$，不难得到递推式：

$$
\begin{equation}
P_{m, n} = \sum_{i=1}^n p^{i-1}(1-p)P_{m-i, n}
\end{equation}
$$

考虑数列$\{P_{m, n}\}_m$的**生成函数**：

$$
\begin{aligned}
G_n(x) &= \sum_{i=0}^{\infty} P_{i, n} x^{i}\\
&=\sum_{i=1}^{\infty} P_{i, n} x^{i}\quad (\text{for } n\geq 1) \\
\end{aligned}\tag{2}
$$

综合$(1)$和$(2)$可以得到：

$$
\begin{aligned}
G_n(x) [1-(1-p)x-p(1-p)x^2-\cdots-p^{n-1}(1-p)x^n] = P_{n, n} x^n = p^{n}x^n
\end{aligned}\tag{3}
$$

于是，

$$
G_n(x) \left[1 - (1-p) x \dfrac{1-p^{n}x^{n}}{1-px}\right] = p^nx^n
$$

$$
G_n(x) = \boxed{\dfrac{(1-px)p^nx^n}{1-x+(1-p)p^{n}x^{n+1}}} \tag{4}
$$

所以使得连续$n$次出现正面需要的投掷次数期望为$E_n = G'_n(1) = \boxed{\dfrac{1-p^n}{p^n(1-p)} }$

方差则为

$$
\begin{aligned}
D_n &= G''(1)+G'(1)-[G'(1)]^2 \\
&= \dfrac{2[(n+1)p^{n+1}-p^{2n+1}+p^{2n}-(n+2)p^n+1]}{p^{2n}(1-p)^2}+\dfrac{1-p^n}{p^n(1-p)}-\left[\dfrac{1-p^n}{p^n(1-p)}\right]^2\\
&=\boxed{\dfrac{1 - (2n+1)(1-p)p^n-p^{2n+1}}{p^{2n}(1-p)^2}}
\end{aligned}
$$

:::info 你问咋算这么复杂的导数？
出门右转[Wolfram Alpha](https://www.wolframalpha.com/)，或者使用本地的Mathematica/Maple/SymPy之类的符号计算软件。
:::

对于$p=\dfrac{1}{2}$，有$G_n(x) = \dfrac{(2-x)x^n}{2^{n+1}(1-x)+x^{n+1}}$，$E_n=2^{n+1}-2$，$D_n = 2^{2n+2} - (2n+1)2^{n+1} - 2$

:::info 分析
也就是说如果硬币公平的话，平均每30把就会出现4连先手**和**4连后手。不过结果的标准差基本上和期望一样大，所以结果很不稳定，所以可能没啥参考价值？
:::

## 抛m次硬币连续至少n次正面的概率

比起期望，我们当然对概率本身更加感兴趣，不过这计算就繁琐了不少。

### 闭形式

记抛m次硬币出现的最长连续正面次数为$Z_m$，那么我们要求的就是$P(Z_m\geq n)$的大小。

沿用上一节的记号，我们有

$$
P(Z_m\geq n) = \sum_{i = 0}^m P_{i, n} \tag{5}
$$

也就是说，所求的就是数列$\{P_{m, n}\}_m$的前m项和。由于前面已经算出了$\{P_{m, n}\}_m$的生成函数，我们可以直接写出$P(Z_m\geq n)$的生成函数$F_n(x)$

$$
\begin{align*}
F_n(x) &= \dfrac{1}{1-x}G_n(x) \\
&= \dfrac{(1-px)p^nx^n}{(1-x)[1-x+(1-p)p^{n}x^{n+1}]} \\
&= \dfrac{1}{1-x} + \dfrac{p^nx^n-1}{1-x+(1-p)p^{n}x^{n+1}} \tag{6}
\end{align*}
$$

:::tip 定义1. （系数算子） $[x^n]f(x)$

对幂级数$\displaystyle \sum_{i=0}^{\infty} a_ix^i$以及$n\in \mathbb{Z}$ 定义

$$
[x^n]\displaystyle \sum_{i=0}^{\infty} a_ix^i = \begin{dcases}
  a_n &,if\ n\geq 0,  \\
  0 &,if\ n<0.
\end{dcases}
$$

按照定义易得以下性质：

- $[x^n]f(x)=\dfrac{f^{(n)}(0)}{\Gamma(n+1)},\quad if\ n\geq 0$
- $[x^n]x^mf(x) = [x^{n-m}]f(x)$
  :::

按照生成函数的定义，我们有

$$
\begin{align*}
P(Z_m\geq n) &= [x^m]F_n(x) = [x^m] \left(\dfrac{1}{1-x} + \dfrac{p^nx^n-1}{1-x+(1-p)p^{n}x^{n+1}}\right)\\
&= [x^m] \left(\sum_{i=0}^{\infty}x^i+(p^nx^n-1)\sum_{i=0}^{\infty}\left[x-(1-p)p^nx^{n+1}\right]^i\right)\\
&= 1 + [x^m](p^nx^n-1)\sum_{i=0}^{\infty}\left[x-(1-p)p^nx^{n+1}\right]^i\\
&=1+\left(p^n[x^{m-n}] - [x^m]\right)\sum_{i=0}^{\infty}x^i\left[1-(1-p)p^nx^{n}\right]^i\\
&=1+\left(p^n[x^{m-n}] - [x^m]\right)\sum_{i=0}^{\infty}x^i\sum_{j=0}^i \binom{i}{j}  (-1)^j (1-p)^j (px)^{nj}\\
&=1+\sum_{i=0}^{m}\left(p^n[x^{m-n-i}] - [x^{m-i}]\right)\sum_{j=0}^i \binom{i}{j}  (-1)^j (1-p)^j (px)^{nj}\\
&=1+p^n\sum_{j=0}^{\lfloor \frac{m}{n} \rfloor - 1 } \binom{m-n(j+1)}{j}(p-1)^jp^{nj} - \sum_{j=0}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j}(p-1)^jp^{nj}\\
&=1+\sum_{j=0}^{\lfloor \frac{m}{n} \rfloor - 1 } \binom{m-n(j+1)}{j}(p-1)^jp^{n(j+1)} - 1 - \sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j}(p-1)^jp^{nj}\\
&=\sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j-1}(p-1)^{j-1}p^{nj} - \sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j}(p-1)^jp^{nj}\\
&=\sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j-1}(p-1)^{j-1}p^{nj}\left[1-\dfrac{m-(n+1)j + 1}{j}(p-1)\right]\\
&=\boxed{\sum_{j=1}^{\lfloor \frac{m}{n} \rfloor } \binom{m-nj}{j-1}(p-1)^{j-1}p^{nj}\left[p+\dfrac{(1+m-nj)(1-p)}{j}\right]} \tag{7}
\end{align*}
$$

### 渐进分析

从闭形式中很难看出来概率的变化趋势，为此我们还需要一个渐进估计表达式。

这一节我们将得到：当$m\gg n$且$n$充分大时，有

$$
P(Z_m \geq n) \sim 1-\mathrm{e}^{(m+1)(p-1)p^n} \tag{8}
$$

#### 亚纯估计定理

道理上来讲，生成函数实际上只是**形式幂级数**，是否收敛并不重要。但如果我们知道它是收敛乃至解析的，那么就能用上一些分析学的方法来对其系数做出估计。

:::tip 定理：亚纯生成函数的估计
设$f(z)$是一个**亚纯函数**，它在包含原点的区域$\mathfrak{R}$内除了有限个极点外是解析的.

设$R>0$是模最小极点的模，$z_0, \cdots, z_s$是$f(z)$的所有模为$R$的极点，$R'>R$是$f(z)$次小模极点的模。

那么对$\forall \varepsilon > 0$，有

$$
[z^n]f(z)=[z^n]\left(\sum_{j=0}^s pp(f;z_j)+O\left(\left(\dfrac{1}{R'}+\varepsilon\right)^n\right)\right)
$$

其中$pp(f;z_j)=\displaystyle \sum_{j=1}^r \dfrac{a_{-j} }{(z-z_j)^j}$为$f$在$r$阶极点$z_j$处**Laurent展开的主要部分**。

:::

这个定理在《发生函数论》的第5章可以找到，这里我们就不加证明直接拿来了。这个定理表明，决定函数系数的最主要因素是它的模最小极点。

进一步，如果$z_0$是$f(z)$的（其中一个）模最小极点，且它是$r$阶极点，那么它对系数的贡献为

$$
\begin{align*}
[x^n]pp(f;z_0) &= [x^n]\sum_{j=1}^r \dfrac{a_{-j}}{(z-z_0)^j} = [x^n]\sum_{j=1}^r \dfrac{(-1)^ja_{-j}}{z_0^j(1-\dfrac{z}{z_0})^j} \\
&=[x^n]\sum_{j=1}^r \dfrac{(-1)^ja_{-j}}{z_0^j}\sum_{k = 0}^{\infty} \binom{k+j-1}{k}\left(\dfrac{z}{z_0}\right)^k\\
&=[x^n]\sum_{k = 0}^{\infty}z^k\left(\sum_{j=1}^r \dfrac{(-1)^ja_{-j}}{z_0^{k+j}}\binom{k+j-1}{j-1}\right)\\
&=\sum_{j=1}^r \dfrac{(-1)^ja_{-j}}{z_0^{n+j}}\binom{n+j-1}{j-1}
\end{align*}
$$

特别地，如果$z_0$是1阶极点，那么它的贡献就是$-\dfrac{Res[f(z), z_0]}{z_0^{n+1}}$.

#### 分析极点阶数

借由亚纯估计定理，我们的思路就是分析生成函数的极点。

回到$P(Z_m \geq n)$的生成函数$(6)$，由于$\dfrac{1}{1-x}$只是单纯给各项系数加上个$1$，我们只需考虑

$$
Q_n(x) = \dfrac{1-p^nx^n}{1-x+(1-p)p^{n}x^{n+1}}
$$

:::info NOTE
$Q_n(x) = \dfrac{1}{1-x}-F_n(x) = \displaystyle \sum_{i=0}^\infty \left[1-P(Z_m \geq n)\right]x^i=\sum_{i=0}^\infty P(Z_m < n) x^i$，因此$Q_n(x)$就是$P(Z_m < n)$的生成函数，
:::

为了得到$Q_n(x)$的极点，我们考虑方程

$$
\varphi(x)\overset{def}{=}1-x+(1-p)p^{n}x^{n+1} = 0 \tag{9}
$$

的根。

首先不难发现$\varphi\left(\dfrac{1}{p}\right)=0$，但$x=\dfrac{1}{p}$是个可去奇点，并不符合要求。

进一步分析，来求极值：

$$
\varphi'(x) = (n+1)(1-p)p^nx^n-1
$$

极值点$x_m = \dfrac{1}{p}\sqrt[n]{\dfrac{1}{(n+1)(1-p)} }$，极小值

$$
\begin{align*}
\varphi(x_m) &=1-\sqrt[n]{\dfrac{1}{p^n(1-p)}\dfrac{n^n}{(n+1)^{n+1}}}\\
&=1-\sqrt[n]{\dfrac{1}{\left(\dfrac{p}{n}\right)^n(1-p)}\dfrac{1}{(n+1)^{n+1}}}\\
&\leq 1- \sqrt[n]{\dfrac{1}{\left[\left(\underbrace{\dfrac{p}{n}+\cdots+\dfrac{p}{n}}_{n}+1-p\right)/(n+1) \right]^{n+1} }\dfrac{1}{(n+1)^{n+1}}}\\
&=0
\end{align*}
$$

最后采用了均值不等式来放缩，当且仅当$p = \dfrac{n}{n+1}$时取等，此时$x_m=1+\dfrac{1}{n}$.

借助均值不等式，不难发现$1 + \dfrac{1}{n} \leq x_m$以及$\varphi(1+(1-p)p^n)>0$，$\varphi(1+\dfrac{1}{n})\leq 0$，因此方程$\varphi(x) = 0$在$(1, +\infty)$上有两个根， 它们分布在$1+1/n$的两侧， 其中一个是$1/p$。

:::details $\varphi(x)=0$的最小模根是实根
~~证明留作习题~~ 不是重点，不想啰嗦
![](https://s2.loli.net/2024/04/28/JbIXjQSD8swxiEe.png)
贴一张$n=8, p=0.6$时$\varphi(x)$在复平面上的函数值作为参考吧~

实际上其它复根的模长不小于另一个实根（提示：使用三角不等式），因此为了使用亚纯估计定理，我们只需要考虑这个函数的实根就足够了。
:::

##### (I) $p\neq\dfrac{n}{n+1}$ 时

$$
\begin{aligned}
Res[Q_n(x); x_0] &= \lim_{x\to x_0} \dfrac{1-p^nx_0^n}{\varphi'(x_0)}\\
&=\dfrac{1-p^nx_0^n}{(n+1)(1-p)p^nx_0^n-1}
\end{aligned}
$$

由亚纯估计定理可知：

$$
\begin{aligned}
P(Z_m < n) &= [x^m]Q_n(x)\\
&\sim \dfrac{1-p^nx_0^n}{1-(n+1)(1-p)p^nx_0^n} \left(\dfrac{1}{x_0}\right)^{m+1}\\
&= \dfrac{1-px_0}{(1-p)(n+1-nx_0)} \left(\dfrac{1}{x_0}\right)^{m+1}
\end{aligned} \tag{10a}
$$

##### (II) $p=\dfrac{n}{n+1}$ 时

$$
a_{-1}=\lim_{x\to x_0} \dfrac{(1-p^nx^n)(x-x_0)}{\varphi(x)} = -\dfrac{2}{(n+1)(1-p)} = -2
$$

$$
a_{-2}=\lim_{x\to x_0}\dfrac{(1-p^nx^n)(x-x_0)^2}{\varphi(x)} = \dfrac{2(1-p^nx_0^n)}{n(n+1)(1-p)p^nx_0^{n-1}} = 0
$$

因此，

$$
\begin{aligned}
P(Z_m < n) &= [x^m]Q_n(x) \sim 2 \left(\dfrac{1}{x_0}\right)^{m+1} 
\end{aligned} \tag{10b}
$$

#### 估计极点$x_0$

当$n$充分大时，总是有$p<\dfrac{n}{n+1}$，这时$1/p$不是$\varphi(x)=0$最小的实根。记最小实根为$x_0$. 注意到$\varphi\left(1+(1-p)p^n\right)>0$和$\varphi\left(1+e(1-p)p^n\right)<0$，结合前面的导数分析，我们有$1+(1-p)p^n<x_0<1+e(1-p)p^n$，因此

$$
x_0=1+\Theta\left((1-p)p^n\right)
$$

进一步，记$K=\displaystyle \lim_{n\to \infty} \dfrac{x_0-1}{(1-p)p^n} \in [1, e]$，则

$$
\begin{align*}

K&=\lim_{n\to \infty} \dfrac{x_0-1}{(1-p)p^n} =\lim_{n\to \infty}x_0^{n+1} \\
&\leq \lim_{n\to \infty} \left[1+e(1-p)p^n\right]^{n+1}\\
&= \exp\left(\lim_{n\to \infty} \ln(1+e(1-p)p^n)(n+1)\right)\\
&\leq \exp\left(\lim_{n\to \infty} e(1-p)p^n(n+1)\right)\\
&=\exp(0) = 1
\end{align*}
$$

因此，$K=1$，也就是说

$$
x_0 \sim 1+(1-p)p^n \quad (n\to \infty)
$$

将以上近似回代到$(10a)$中，有

$$
\begin{align*}
P(Z_m < n) &\sim \dfrac{1-px_0}{(1-p)(n+1-nx_0)} \left(\dfrac{1}{x_0}\right)^{m+1} \\
&\sim \left(\dfrac{1}{x_0}\right)^{m+1}\\
&= \left(\dfrac{1}{1+ (1-p)p^n}\right)^{m+1}\\
&=\left(1+ \dfrac{1-p}{p^{-n}}\right)^{-(m+1)}\\
&=\left(\left(1+ \dfrac{1-p}{p^{-n}}\right)^{p^{-n}}\right)^{-p^n(m+1)}\\
&\sim \left(\mathrm{e}^{1-p}\right)^{-p^n(m+1)}\\
&= \boxed{\mathrm{e}^{(m+1)(p-1)p^n}}
\end{align*}
$$

从而，

$$
\boxed{P(Z_m \geq n) = 1 - P(Z_m < n) \sim 1 - \mathrm{e}^{(m+1)(p-1)p^n} }\tag{8}
$$

> $\mathrm{e}$：又是我~

:::warning TODO: 误差分析
好麻烦欸，有空再整吧...~~（多半是鸽了）~~
:::

## Bo∞中的连胜

:::info Bo3决定胜负的卡牌游戏

游戏王实卡的赛制是Bo3，也就是三局两胜制。在前一局游戏结束后，败方可以选择下一局的先后手，同时双方可以根据对方的卡组以及先后手情况从自己预先准备的15张备牌中更换一些针对卡。

由于换备和三局两胜制的存在，因此会比MD的一把定胜负更平衡一些。

:::

设想这样一种赛制： **双方无限对局，当某一方首先获得单局2连胜时视为游戏胜利。** 理论上来说，相比Bo3赛制，这样会要求更大的玩家水平和卡组构筑差距才能决出胜负。

假设对局双方分别使用卡组A和卡组B，在第一局（主卡局），卡组A对卡组B的先攻胜率为$p_0$；在备牌局，卡组A的先攻胜率为$p_1$，后攻胜率为$p_2$.

为了简单起见，我们**假设双方卡组都是先手比后手更有优势的卡组**。因此当某个玩家输了一局的时候，下一局他一定会选择先手。

那么问题是，如果使用卡组A的玩家先攻：

- 游戏期望在第几局结束？
- 他的最终胜率是多少？

### 期望

这个对局过程显然是**马尔可夫**的。

对局总共有5种状态，$S=\{S_0=-2, S_1 = -1, S_2 = 0, S_3 = 1, S_4 = 2\}$，其中$S_2=0$是第一局（初始态），$S_0$和$S_4$分别代表连输/连赢2次的状态（吸收态）。

我们可以写出相应的转移矩阵：

$$
\mathcal{M}=\begin{bmatrix}
 1 & 0 & 0 & 0 & 0\\
 1 - p_1 & 0 & 0 & p_1 & 0 \\
 0 & 1-p_0 & 0 & p_0 & 0 \\
 0 & 1-p_2 & 0 & 0 & p_2 \\
 0 & 0 & 0 & 0 & 1
\end{bmatrix}
$$

设从状态$S_i$转移到某个吸收态的期望为$E_i$，则

$$
\begin{cases}
E_0=0\\
E_1=(1-p_1)E_0+p_1E_3+1\\
E_2=(1-p_0)E_1+p_0E_3+1\\
E_3=(1-p_2)E_1+p_2E_4+1\\
E_4=0
\end{cases}
$$

解得，$(E_1,\ E_2,\ E_3) = \dfrac{\left(1+p_1,\ 2+p_0+p_1p_2-p_0(p_1+p_2),\ 2-p_2\right)}{1-p_1+p_1p_2}$

也就是说，分出胜负的期望局数是$E_2=\dfrac{2+p_0+p_1p_2-p_0(p_1+p_2)}{1-p_1+p_1p_2}$

### 概率

设对局中前一把赢了之后最终取胜的概率为$A$，前一把输了之后最终取胜的概率为$B$，利用

$$
\begin{cases}
A=p_2+(1-p_2)B\\
B=p_1A
\end{cases}
$$

解得，$(A, B) = \left(\dfrac{p_2}{1-p_1+p_1p_2},\dfrac{p_1p_2}{1-p_1+p_1p_2}\right)$

最终胜率为$P = p_0A+(1-p_0)B = \dfrac{p_2(p_0+p_1-p_0p_1)}{1-p_1+p_1p_2}$

### K连胜的情形

之前我们讨论的是2连胜的情形，假如将胜利条件改为有一方$K$连胜才结束那又如何计算呢？

#### 游戏局数的期望

期望的计算还是一样，根据状态转移矩阵可以列出$2K+1$元的线性方程组：

$$
\begin{dcases}
  E_i=(1-p_2)E_{-1}+p_2E_{i+1}+1 &,i\in \{1,\cdots,K-1\}\\
  E_i=p_1E_1+(1-p_1)E_{i-1}+1 &,i \in \{-1, \cdots, -K+1\}\\
  E_0=(1-p_0)E_{-1}+p_0E_1+1\\
  E_K=E_{-K}=0
\end{dcases}
$$

由于我们只关心$E_0$，简单化简后可以得到

$$
\begin{dcases}
  E_1=\left(E_{-1}+\dfrac{1}{1-p_2}\right)\left(1-p_2^{K-1}\right) \\
  E_{-1}=\left(E_{1}+\dfrac{1}{p_1}\right)\left(1-(1-p_1)^{K-1}\right)\\
  E_0=(1-p_0)E_{-1}+p_0E_1+1
\end{dcases}
$$

具体的解就不写了，太长了😵

关于胜率的求解，我们可以扩展之前的方法，但更优雅的做法是借助**鞅论**的知识。

#### 鞅与停时定理

:::tip 定义2. 鞅

对于随机过程$\{X_n\}$和$\{M_n\}$。如果满足：

- $M_n$只与$X_0, \cdots, X_n$有关，
- $E(M_{n+1}|X_0, \cdots, X_n) = M_n$

则称$\{M_n\}$是一个关于$\{X_n\}$的**鞅**。

其中条件2也可以等价地写成$E(M_{n+1}-M_n|X_0, \cdots, X_n) = 0$，如果将等号改为大于等于/小于等于，那么过程$\{M_n\}$就称为**上鞅**/**下鞅**。

> 上鞅和下鞅的定义也有反过来的，可能是因为和函数凹凸性相关，于是和凹函数/凸函数一样成了笔糊涂账。Anyway，这里我们只用鞅，并不涉及上鞅与下鞅。

:::

直观地来讲，如果一个随机过程它每一步的期望都保持恒定不变，是一个平稳的过程，那么就称它为鞅。换句话说，对任意有限的$n$，我们有

$$
E(M_{n}) = E(M_{0})
$$

若$T$是$\{M_n\}$的一个**停时**，记$T\wedge n = \min\{T, n\}$，那么显然有

$$
E(M_{T\wedge n}) = E(M_{0})
$$

停时定理所讨论的是：在什么情况下有$E(M_T) = E(M_0)$成立？

:::tip 鞅停时定理（Optional Stopping Theorem）

$\{M_n\}$是关于$\{X_n\}$的**鞅**，$T$是$\{M_n\}$的一个**停时**，那么以下均为$E(M_T) = E(M_0)$的充分条件：

1. 存在$K<\infty$，使得$P(T\leq K)=1$
2. $E(T)<\infty$，并且$E(|M_{n+1}-M_n|)\leq c < \infty$
3. $P(T\leq \infty)=1$，并且$M_{T\wedge n} \leq K < \infty$

> [!note]
> OST的成立条件其实就是在问$M_{T\wedge n}$是否**一致可积**，也就是能否**交换积分和极限的顺序**。如果答案是肯定的，那么显然就有
> $$E(M_{0}) = \lim_{n\to\infty}E(M_{T\wedge n})=E\left(\lim_{n\to\infty}M_{T\wedge n}\right)=E(M_T)$$

:::

#### 计算思路

那么如何使用OST来计算$K$连胜的概率呢？

我们设随机变量$X_n=\{\text{当前连胜场次}\}$，如果当前连胜那么$X_n$为正，反之则$X_n$为负，$T$为首次$K$连胜或$K$连败所需的局数，$K$连胜的概率为$P^+_{K}$.

计算的思路是构造一个关于$\{X_n\}$鞅$\{f(X_n)\}$，这个鞅满足OST的使用条件，这样一来我们有

$$
f(X_0)=E(f(X_0))=E(f(X_T))=P^+_{K}f(K)+(1-P^+_{K})f(-K)
$$

从而

$$
P^+_{K}=\dfrac{f(X_0)-f(-K)}{f(K)-f(-K)} \tag{*}
$$

所以问题的关键就在于如何构造这个$\{f(X_n)\}$

#### 构造鞅函数

本来这时候该**注意到**了，但本文的目的并不是显摆注意力，所以这里就写明白如何切实地构造这个函数。

对$X_n>0$，也就是连胜中的情形，此时赢一把$X_{n+1}=X_n+1$，输一把$X_{n+1}=-1$，按照鞅的定义，我们写出

$$
f(X_n)=E(f(X_{n+1})|X_n)=p_2f(X_n+1)+(1-p_2)f(-1)
$$

对$X_n<0$，也就是连败中的情形，此时赢一把$X_{n+1}=1$，输一把$X_{n+1}=X_n-1$，按照鞅的定义，我们写出

$$
f(X_n)=E(f(X_{n+1})|X_n)=p_1f(1)+(1-p_1)f(X_n-1)
$$

对$X_n=0$，这是第一局主卡局，有

$$
f(0)=E(f(X_1)| X_0=0)=p_0f(1)+(1-p_0)f(-1)
$$

为了方便，令$f(0)=0$，我们只需求解函数方程组：

$$
\begin{dcases}
  f(n)=p_2f(n+1)+(1-p_2)f(-1) &,n>0\\
  f(n)=p_1f(1)+(1-p_1)f(n-1) &,n<0\\
  f(0)=p_0f(1)+(1-p_0)f(-1)=0,
\end{dcases}
$$

这并不困难：

$$
f(n)=\begin{dcases}
  p_2^{-n+1}-p_0 &,n>0\\
  0 &,n=0\\
  -(1-p_1)^{n+1}+1-p_0 &,n<0
\end{dcases}
$$

容易验证$f(n)$是有界的，因而可以使用停时定理。

于是由$(*)$，我们得到

$$
\begin{align*}
P^+_{K}&=\dfrac{-f(-K)}{f(K)-f(-K)}\\
&=\dfrac{(1-p_1)^{-K+1}-1+p_0}{p_2^{-K+1}-p_0+(1-p_1)^{-K+1}-1+p_0}\\
&=\boxed{\dfrac{p_2^{K-1}\left[1-(1-p_0)(1-p_1)^{K-1}\right]}{(1-p_1)^{K-1}+p_2^{K-1}-(1-p_1)^{K-1}p_2^{K-1}}} \tag{11}
\end{align*}
$$

## 天梯升降段

另一个有趣的问题是已知卡组的胜率为$p$，如何计算在MD天梯升段/降段的期望局数和概率？

> 你可以考虑先攻胜率$p_1$后攻胜率$p_2$，但无非就是把每步转移概率里的$p$换成$(p_1+p_2)/2$

MD的段位机制如下：

- 累计净胜$K$局升段（白金/钻石段$K=4$，大师段$K=5$）
- 净胜0局的情况下连败3局掉段
- 掉段流程中只要有赢一局，回到净胜1局的状态

这个问题的解法和之前的$K$连胜问题基本一样：
* 根据状态转移矩阵列出线性方程组求解期望
* 构造鞅使用停时定理来计算最终概率

读者可以参考上一节的计算，这里就简单写写。

### 钻石段：$K=4$

状态转移矩阵：

$$
\mathcal{M_p}=\begin{bmatrix}
 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0\\
 1-p & 0 & 0 & 0 & p & 0 & 0 & 0\\
 0 & 1-p & 0 & 0 & p & 0 & 0 & 0\\
 0 & 0 & 1-p & 0 & p & 0 & 0 & 0\\
 0 & 0 & 0 & 1-p & 0 & p & 0 & 0\\
 0 & 0 & 0 & 0 & 1-p & 0 & p & 0\\
 0 & 0 & 0 & 0 & 0 & 1-p & 0 & p\\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1
\end{bmatrix}
$$

记$\mathbf{E}=\begin{pmatrix}
  E_{-3}\\
  E_{-2}\\
  E_{-1}\\
  E_{0}\\
  E_{1}\\
  E_{2}\\
  E_{3}\\
  E_{4}
\end{pmatrix}$，$\mathbf{c}=\begin{pmatrix}
  0\\
  1\\
  1\\
  1\\
  1\\
  1\\
  1\\
  0
\end{pmatrix}$，$\mathbf{A}=\begin{pmatrix}
  2& &\\
  & \mathbf{I}_{6} &\\
  & & 2
\end{pmatrix}$

解线性方程$(\mathbf{A}-\mathcal{M_p})\mathbf{E}=\mathbf{c}$，得

$$
\mathbf{E}=\dfrac{1}{p^6-5p^5+11p^4-13p^3+11p^2-5p+1}\begin{pmatrix}
  0\\
  2p^3+2p^2-p+1 \\
  -2p^4+2p^3+5p^2-3p+2 \\
  2p^5-4p^4-p^3+10p^2-6p+3 \\
  -p^5+5p^4-11p^3+15p^2-9p+4 \\
  p^5-2p^4-3p^3+13p^2-12p+5 \\
  -2p^5+12p^4-29p^3+36p^2-22p+6 \\
  0
\end{pmatrix}
$$

为了计算升段概率，我们构造$f(n)$使$\{f(X_n)\}$是个鞅，

$$
f(n)=\begin{dcases}
  \dfrac{p}{2p-1}\left[1-\left(\dfrac{1-p}{p}\right)^n\right] &,n\geq 0\\
  1-(1-p)^n &,n<0
\end{dcases}
$$

停时$T=\{ \text{\small 首次到达} n=4 \vee n=-3 \}$，设升段概率为$p^+$，则由停时定理

$$
f(0)=E(f(X_T))=p^+f(4)+(1-p^+)f(-3)=0
$$

解得，

$$
p^+=\dfrac{-f(-3)}{f(4)-f(-3)}=\dfrac{p^4(p^2-3p+3)}{p^6 - 5 p^5 + 11 p^4 - 13 p^3 + 11 p^2 - 5 p + 1}
$$

### 大师段：$K=5$

状态转移矩阵：

$$
\mathcal{M_p}=\begin{bmatrix}
 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0  & 0\\
 1-p & 0 & 0 & 0 & p & 0 & 0 & 0& 0\\
 0 & 1-p & 0 & 0 & p & 0 & 0 & 0& 0\\
 0 & 0 & 1-p & 0 & p & 0 & 0 & 0& 0\\
 0 & 0 & 0 & 1-p & 0 & p & 0 & 0& 0\\
 0 & 0 & 0 & 0 & 1-p & 0 & p & 0& 0\\
 0 & 0 & 0 & 0 & 0 & 1-p & 0 & p& 0\\
 0 & 0 & 0 & 0 & 0 & 0 & 1-p & 0 & p\\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0& 1
\end{bmatrix}
$$

解线性方程$(\mathbf{A}-\mathcal{M_p})\mathbf{E}=\mathbf{c}$，得

$$
\mathbf{E}=\dfrac{\begin{pmatrix}
  0\\
  3p^4 + 3p^2 - 2p + 1\\
  3p^5 - 6p^4 + 3p^3 - 8p^2 + 5p - 2\\
  3p^6 - 9p^5 + 12p^4 - 11p^3 + 16p^2 - 9p + 3\\
  -2p^5 + 10p^4 - 19p^3 + 24p^2 - 13p + 4\\
  3p^6 - 14p^5 + 29p^4 - 35p^3 + 32p^2 - 17p + 5\\
  -p^5 + 8p^4 - 22p^3 + 32p^2 - 21p + 6\\
  3p^6 - 19p^5 + 52p^4 - 78p^3 + 69p^2 - 33p + 7\\
  0
\end{pmatrix}}{2p^6 - 10p^5 + 22p^4 - 24p^3 + 16p^2 - 6p + 1}
$$

鞅和之前的一样（因为段位机制没变，只是停止条件变了），停时变成$T=\{ \text{\small 首次到达} n=5 \vee n=-3 \}$，此时

$$
p^+=\dfrac{-f(-3)}{f(5)-f(-3)}=\dfrac{(p^2 - 3p + 3)p^5}{2p^6 - 10p^5 + 22p^4 - 24p^3 + 16p^2 - 6p + 1}
$$

### 简单分析

#### 胜率

我们把$K=4$和$K=5$时的胜率-升段率曲线画出来：

![胜率-升段率曲线](https://s2.loli.net/2024/04/30/Fb79XOQapz8qPGJ.png)

* 一个卡组要在天梯上保持不升不降，其胜率并不需要达到$50\%$：在白金/钻石段只需要$46\%$的胜率，而在大师段则需要$48\%$
* 一个单局胜率在$50\%$的卡组实际上有$60\%$左右的概率是能升段的

#### 游戏局数

把$K=4$和$K=5$时的胜率-期望局数曲线画出来：

![胜率-期望局数曲线](https://s2.loli.net/2024/04/30/SAXjKg25fiaDYP1.png)

* 除非单局胜率特别高（$>78\%$），不然大师段升降段会比钻石段更快（大概是因为掉段更容易？）
* 期望局数在略小于$50\%$胜率的地方取到最大（不过并不是总体胜率$50\%$的点，还要更小一点）
* 期望上来说，大师段10把升降段，钻石白金13把升降段

### 一般情况

总结一下计算结果，如果某小段需要累计$N$胜升段，0胜场$M$连败掉段，卡组单局胜率为$p$。

#### 升段率

由于段位机制没有变化，因此对应的鞅$\{f(X_n)\}$也不变：

$$
f(X_n)=\begin{dcases}
  \dfrac{p}{2p-1}\left[1-\left(\dfrac{1-p}{p}\right)^{X_n}\right] &,X_n\geq 0\\
  1-(1-p)^{X_n} &,X_n<0
\end{dcases}
$$

于是能成功升段的概率就是：

$$
p^+=\dfrac{-f(-M)}{f(N)-f(-M)}
$$

#### 期望

升降段发生所需的期望局数由线性方程$(\mathbf{A}-\mathcal{M}_p)\mathbf{E}=\mathbf{c}$给出。其中，

$$
\begin{gather*}
  \mathbf{A}=\begin{pmatrix}
  2& &\\
  & \mathbf{I}_{N+M-1} &\\
  & & 2
\end{pmatrix},\mathbf{c}=\begin{pmatrix}
  0\\
  \mathbf{1}_{N+M-1}\\
  0
\end{pmatrix},\\
\mathcal{M}_p=\begin{bmatrix}
    1   &     &        &     &     &        &     &        &   &   \\
    q &     &        &     &     & p      &     &        &   &   \\
        & q &        &     &     & p      &     &        &   &   \\
        &     & \ddots &     &     & \vdots &     &        &   &   \\
        &     &        & q &     & p      &     &        &   &   \\
        &     &        &     & q &        & p   &        &   &   \\
        &     &        &     &     & \ddots &     & \ddots &   &   \\
        &     &        &     &     &        & q &        & p &   \\
        &     &        &     &     &        &     & q    &   & p \\
        &     &        &     &     &        &     &        &   & 1
\end{bmatrix},\ q=1-p 
\end{gather*}
$$

解得的列向量$\mathbf{E}$中的每一个元素代表从某个胜负场状态开始到达升段/降段的期望局数。

当然，也可以把上述矩阵形式的线性方程组写成大家更熟悉的形式：

$$
\begin{dcases}
  E_i=(1-p)E_{i-1}+pE_{i+1}+1 &,i\in \{0,1,\cdots,N-1\}\\
  E_i=(1-p)E_{i-1}+pE_1+1 &,i \in \{-1, \cdots, -M+1\}\\
  E_N=E_{-M}=0
\end{dcases}
$$

如果我们只关心$E_0$，那么和上一节类似，可以直接化简上面的线性方程组：

$$
\begin{dcases}
  E_1=\dfrac{N-1}{2p-1}+\left(E_0-\dfrac{N}{2p-1}\right)\dfrac{p^N(1-p)-p(1-p)^N}{p^{N+1}-p(1-p)^N}\\
  E_0=\left(E_1+\dfrac{1}{p}\right)\left[1-(1-p)^M\right]
\end{dcases}
$$

:::tip 那么是咋化简的呢

实际上就是将前两行当作数列的递推方程来处理，转化为求数列通项的问题。

不过这里这个比上一节里的更难些，上一节中两个递推方程都是一阶线性递推，只需凑成等比数列就行。这里第一个方程是二阶线性递推，会稍麻烦些。这里还是写写详细过程吧。

首先来处理比较简单的第二行，利用待定系数法消去常数1，凑成$E_i+\lambda = (1-p)[E_{i-1}+\lambda]$的形式：

$$
E_i-\left(E_1+\dfrac{1}{p}\right)=(1-p)\left[E_{i-1}-\left(E_1+\dfrac{1}{p}\right)\right],\quad i\in\{0, -1, \cdots, -M+1\}
$$

这就是等比数列的递推关系，考虑$E_{-M}=0$，我们就得到

$$
E_0=\left(E_1+\dfrac{1}{p}\right)\left[1-(1-p)^M\right]
$$

再来处理第一行。首先还是要消去常数项，但是因为$E_i$前系数刚好相互抵消的关系，直接用$E_i+\lambda$来凑行不通，换用$E_i+\lambda i$，于是：

$$
E_i+\dfrac{i}{2p-1}=(1-p)\left[E_{i-1}+\dfrac{i-1}{2p-1}\right]+p\left[E_{i+1}+\dfrac{i+1}{2p-1}\right],\quad i\in{0, 1, \cdots, N-1}
$$

设$A_i = E_i+\dfrac{i}{2p-1}$，我们有$A_i = (1-p)A_{i-1}+pA_{i+1}$。这是个齐线性二阶递推，其特征方程是$px^2-x+1-p=0$，两个特征根分别为$x_1=1$和$x_2=\dfrac{1-p}{p}$，于是：

$$
A_i = \lambda_1\left(\dfrac{1-p}{p}\right)^i+\lambda_2
$$

考虑$A_{N}=E_{N}+\dfrac{N}{2p-1}=\dfrac{N}{2p-1}$以及$A_0 = E_0$，有

$$
A_i = \dfrac{E_0-\dfrac{N}{2p-1}}{p^N-(1-p)^N}\cdot\dfrac{p^N(1-p)^i-p^i(1-p)^N}{p^i}+\dfrac{N}{2p-1}
$$

代入$i=1$即得

$$
E_1=\dfrac{E_0-\dfrac{N}{2p-1}}{p^N-(1-p)^N}\cdot\dfrac{p^N(1-p)-p(1-p)^N}{p}+\dfrac{N-1}{2p-1}
$$

:::
