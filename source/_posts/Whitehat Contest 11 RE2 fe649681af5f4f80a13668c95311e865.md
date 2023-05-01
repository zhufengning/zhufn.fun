---
title: Whitehat Contest 11 RE2
urlname: Whitehat Contest 11 RE2

date: April 18, 2023 12:14 PM
tags: [go, reverse]
---

我（在reverse题中）最讨厌的语言——go。

被ida里出现的奇怪结构体吓晕。

main函数在cutter里怎么是runtime.text啊（不过在sym.runtime.main里被调用了）。

首先，整个程序被包裹在一个大大的死循环中，然后不断scanf %d。当读到EOF的时候停止，判断是不是13个数字（表面上，实际上是12个）数字。

如果当前数字不是eof，就会被append到数组里（下标甚至从1开始，第0位是0）。还有一个用来统计每个数字出现次数的数组，只能出现一次。

check的过程是这样的，因为中间有一大堆判断数组是否越界的语句（已删），所以容易被搞晕，导致看不到var_1e8h的初值。

```c
var_1e8h = 1;
uVar5 = 1;
do {
    uVar2 = *(uint64_t *)(var_18h + uVar5 * 8);
    var_1e8h = ((&var_168h)[uVar2] + -1) * (&var_1d0h)[var_1e0h - uVar5] + var_1e8h;

    uVar2 = *(uint64_t *)(var_18h + uVar5 * 8);
    while (uVar2 = uVar2 + 1, (int64_t)uVar2 <= var_1e0h) {
        (&var_168h)[uVar2] = (&var_168h)[uVar2] + -1;
    }
    uVar4 = (uint32_t)uVar2;
    uVar5 = uVar5 + 1;
} while ((int64_t)uVar5 <= var_1e0h);
```

换成python就是（写的时候把数组哪个是哪个弄错了，对着汇编改的）

```python
def check(inpt):
    #inpt = [0,6,12,9,4,5,1,2,8,10,7,11,3]
    len_nums = 0xd
    v1d0h = [1,1,2,6,0x18,0x78,0x02d0,0x13b0,0x9d80,0x058980,0x375f00,0x02611500,0x1c8cfc00]

    v18h = []
    v168h = list(range(0xd))

    for i in inpt:
        v18h.append(i)

    i = 1
    s = 1
    while True:
        #print(v168h)
        r11 = v18h
        r8 = r11[i]
        rbx = v168h[r8]-1
        r8=len_nums-1-i
        rbp = v1d0h[r8]
        #print(hex(rbx), hex(rbp))
        s += (rbx*rbp)
        j = v18h[i]+1
        while j<=len_nums-1:
            v168h[j] -= 1
            j+=1
        i+=1
        if i > len_nums -1:
            break
    print(hex(s))
    return s == 0xe37f550
```

之后的正解大概是一个数学问题。[https://dakutenpura.hatenablog.com/entry/2016/06/28/030236](https://dakutenpura.hatenablog.com/entry/2016/06/28/030236)

不过我不会，于是打了暴搜，搜`permutations(range(0,13))`，卡爆了。

但是经过观察发现，s竟然从1开始，每次增加1。于是就变成了：

```python
j = 0
for i in permutations(range(0,13)):
    #sys.stdout.write("%s\n"%str(i))
    j += 1
    if j == 0xe37f550:
        print(check(i))
        print(i)
        break
```
