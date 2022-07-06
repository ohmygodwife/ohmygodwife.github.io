---
layout: post
title: "quad residue二次剩余"
date: 2021-12-08 16:55:05 +0800
description: ""
category: ctf/crypto
tags: []
mathjax: true
---

#### [定义](https://oi-wiki.org/math/number-theory/quad-residue/)

一个数n，如果模p同余于某个数的平方，称n为模p的二次剩余，否则，是p的非二次剩余。对二次剩余求解，也就是对常数n解下面的方程：
$$
x^2=n\mod p
$$
通俗意义上可以认为是求模意义下的开方（有限域开根）。

#### 解的数量

$$
x^2=n\mod p，满足模p的二次剩余的n共有\frac{p-1}{2}个，非二次剩余有\frac{p-1}{2}个（0除外）。\\
证明：显然有(p-x)^2=x^2\mod p，因此，当x\in [1,\frac{p-1}{2}]可以取得所有解。\\
接下来只需要证明当x\in [1,\frac{p-1}{2}]时，x^2\mod p两两不同即可。\\
反证法，假设存在不同的两个整数x,y\in [1,\frac{p-1}{2}]，且x^2=y^2\mod p\\
则：x^2-y^2=0\mod p，(x+y)(x-y)=0\mod p\\
显然：-p<x+y<p,-p<x-y<p,x+y\neq 0,x-y\neq 0，假设不成立，原命题成立。
$$

#### 勒让德符号

$$
\left(\frac{n}{p}\right)=\begin{cases} 1，&p\nmid n且n是p的二次剩余\\
-1，&p\nmid n且n是p的非二次剩余\\
0，&p\mid n\end{cases}
$$

#### 欧拉判别准则

$$
\left(\frac{n}{p}\right)=n^{\frac{p-1}{2}}\mod p（p为奇素数）,即：若n是二次剩余，当且仅当n^{\frac{p-1}{2}}=1\mod p\\
证明：充分性，设n=x^2\mod p，则：n^{\frac{p-1}{2}}=x^{p-1}=1\mod p\\
必要性，设g是模p的一个原根，n=g^k\mod p，那么n^{\frac{p-1}{2}}=g^{\frac{k}{2}(p-1)}=1\mod p\\
因此，\frac{k}{2}(p-1)是\varphi(p)=p-1的倍数，故k是偶数，令x=g^{\frac{k}{2}}可使得n=x^2\mod p成立。
$$

#### Cipolla算法

$$
通过随机法找到一个数a，满足a^2-n是模p的非二次剩余，即(a^2-n)^{\frac{p-1}{2}}=-1\\
类似“复数表达法”，定义i^2=a^2-n，有了a和i后，x^2=n\mod p的解为(a+i)^{\frac{{p+1}}{2}}\\
证明：\\
引理1：(a+b)^p=\sum_{i=0}^pC^i_pa^{p-i}b^i=\sum_{i=0}^p\frac{p!}{(p-i)!i!}a^{p-i}b^i=a^p+b^p\mod p\\
只有i=0和i=p没有因子p，其他因为有因子p模p为0\\
引理2：i^p=i^{p-1}*i=(i^2)^{\frac{p-1}{2}}*i=(-1)*i=-i，因为a^2-n是模p的非二次剩余\\
引理3：费马小定理，a^p=a\mod p\\
x=(a+i)^{\frac{{p+1}}{2}}=((a+i)^{p+1})^{\frac{1}{2}}=((a+i)^p*(a+i))^{\frac{1}{2}}=((a^p+i^p)*(a+i))^{\frac{1}{2}}（引理1）\\
=((a-i)*(a+i))^{\frac{1}{2}}（引理2，3）\\
=(a^2-i^2)^{\frac{1}{2}}=(a^2-(a^2-n))^{\frac{1}{2}}=n^{\frac{1}{2}}\mod n
$$

#### [Tonelli-Shanks](https://blog.csdn.net/wmdcstdio/article/details/49862189) 算法

$$
p-1=q*2^s，q为奇数。令r=n^{\frac{q+1}{2}}，t=n^q\\
则有：r^2=nt\mod p，当t=1\mod p时，r即为所求的开方值\\
选择p的一个非二次剩余z，令c=z^q\\
循环：找到最小的i，使得t^{2^i}=1\mod p\\
令b=c^{2^{s-i-1}},r'=b*r,c'=b^2,t'=c'*t=b^2*t,s=i\\
此时，仍有：r'^2=b^2*r^2=b^2*nt=t'n\mod p。因此，当t'=1\mod p时，r'即为所求的开方值
$$

#### [模数不为奇素数](https://blog.csdn.net/zxyoi_dreamer/article/details/85195819)

$$
\left(\frac{a}{p_1*p_2*...*p_r}\right)=\left(\frac{a}{p_1}\right)\left(\frac{a}{p_2}\right)...\left(\frac{a}{p_r}\right)\\
\left(\frac{a_1*a_2*...*a_r}{p}\right)=\left(\frac{a_1}{p}\right)\left(\frac{a_2}{p}\right)...\left(\frac{a_r}{p}\right)\\
一般二次互反律：\left(\frac{q}{p}\right)=\left(\frac{p}{q}\right)(-1)^{\frac{(p-1)(q-1)}{4}}，p、q是奇素数\\
二次互反律的第一补充：\left(\frac{-1}{p}\right)=(-1)^{\frac{p-1}{2}}=\begin{cases}1,&p\equiv 1\mod 4\\
-1,&p\equiv 3\mod 4\end{cases}\\
二次互反律的第二补充：\left(\frac{2}{p}\right)=(-1)^{\frac{p^2-1}{8}} = \begin{cases}1,&p\equiv 1 or 7 \mod 8\\
-1,&p\equiv 3 or 5\mod 8\end{cases}
$$

