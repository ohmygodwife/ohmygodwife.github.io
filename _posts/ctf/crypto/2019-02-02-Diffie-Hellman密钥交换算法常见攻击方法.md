---
layout: post
title: "Diffie-Hellman密钥交换算法常见攻击方法"
date: 2019-02-02 16:40:22 +0800
description: ""
category: ctf/crypto
tags: []
mathjax: true
---

交换对称加密的密钥，可以通过非对称加密方式传输。或者通过Diffie-Hellman密钥交换算法，与前者不同的是，本方法，交换双方每人约定一个自己的私钥，最后使用的对称加密密钥由约定的两个私钥各自计算得到。

公开的素数p和p的本原单位根（primitive root module p）记为g，可通过sagemath函数计算得到：`g=primitive_root(p)`。原单位根定义为：g的1到p-1次幂模p的值各不相同，即：`g mod p, g^2 mod p,g^(p-1)mod p`组成1到p-1的所有整数。

服务器端私钥和公钥分别是a和A，客户端私钥和公钥分别为b和B

$$
A=g^a\mod p\\B=g^b\mod p\\服务器端得到客户端公钥B后，计算共享密钥K=B^a\mod p=(g^b\mod p)^a=g^{ab}\mod p\\客户器端得到服务端公钥A后，计算共享密钥K=A^b\mod p=(g^a\mod p)^b=g^{ab}\mod p
$$

指数a称为A以g为基数的模p的离散对数（指数），记为：ind_g,p(A)。可见：DH算法的有效性依赖于计算离散对数的难度，即通过公钥难以直接计算私钥。（对比：RSA一般是计算模根）

#### A，g，p较小，离散对数直接计算a

- [Discrete logarithm calculator](https://www.alpertron.com.ar/DILOG.HTM)（结果有时不准确）

- 利用[sympy库](https://blog.csdn.net/qq_43531895/article/details/106108139)：`sympy.ntheory.discrete_log(p, A, g)`

#### p可分解为小素数

[pohlig-hellman算法](https://blog.csdn.net/oampamp1/article/details/104061969)
$$
p=x*y\\g^a=g^{(k*(x-1)+amodxminus1)}=g^{amodxminus1}\mod x\\分别求得amodxminus1和amodyminus1，通过中国剩余定理求得a
$$

#### p为`4k+1`型素数

p=prime * 2 ** bits，可求解离散对数小于bits位的情况，Cipolla算法（sample：strange_p, yiwang-2023）