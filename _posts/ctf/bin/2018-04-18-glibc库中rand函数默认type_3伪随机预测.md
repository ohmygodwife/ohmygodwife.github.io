---
layout: post
title: "glibc库中rand函数默认TYPE_3伪随机预测"
date: 2018-04-18 20:08:23 +0800
description: ""
category: ctf/bin
tags: [glibc, rand]
---

参考： [glibc库中rand函数的实现详解与预测手法](http://www.freebuf.com/articles/web/99093.html)

辅助数组ri：

> (1) r0 = s
>
> (2) ri = (16807 * (signed int) r(i-1)) mod 2147483647 (for i = 1...30)
>
> (3) ri = r(i-31) (for i = 31...33)
>
> (4) ri = (ri-3 + ri-31) mod 4294967296 (for i ≥ 34)

r0…r343 会被丢弃，第一个输出Oi实际上是：

>(5) oi = r(i+344) >> 1

对于i>=31时，输出的oi满足：

> o(i) = r(i+344)>>1 (5)
> 
>       = r(i+344 - 3) + r(i+344 - 31) mod 2^32 >> 1 (4)
>       
>       **= o(i-3)<\<1 + 1/0 + o(i-31)<\<1 +1/0 mod 2^32 >> 1 (5)**
>       
>       = (o(i-3)+o(i-31)+1/0)<\<1 + 1/0 mod 2^32 >>1
>       
>       = (o(i-3)+o(i-31)+1/0)<\<1 mod 2^32 >> 1 + 1/0 mod 2^32 >> 1
>       
>       = o(i-3)+o(i-31)+1/0 mod 2^31 （由于1/0 mod 2^32 >> 1 = 0)

因此

> (6.1) oi = o(i-31) + o(i-3) mod 2\**31, for all i ≥ 31
> 
> (6.2) oi = o(i-31) + o(i-3) + 1 mod 2\**31, for all i ≥ 31

可见，如果可以获取任意连续的31个rand值，即可以推导出下一个rand值。

