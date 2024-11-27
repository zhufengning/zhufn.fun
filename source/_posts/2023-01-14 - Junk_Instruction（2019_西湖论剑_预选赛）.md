---
title: Junk_Instruction（2019_西湖论剑_预选赛）
urlname: Junk_Instruction（2019_西湖论剑_预选赛）

date: January 14, 2023 10:48 PM
tags: [rc4, reverse]
---
题目：xctf

MFC程序，使用xspy查看窗口信息。

check按钮的id为03e9，同时窗口存在`OnCommand: notifycode=0000 id=03e9,func= 0x00C72420(Junk_Instruction.exe+ 0x002420 )`函数。

进入该函数（sub_402420），可以看到有一个条件判断：`if ( (unsigned __int8)sub_402600(v2 + 16) )`。两个分支分别是弹出正确和错误的对话框，说明`sub_402600`是检验函数。

这个函数中就出现了很多垃圾指令了。就是先用一个call压好返回地址，再把栈里的返回地址弹出来，改一下，压回去。啪，返回地址变了。

```
.text:0040293F E8 00 00 00 00                call    $+5
.text:0040293F
.text:00402944
.text:00402944                               loc_402944:                             ; DATA XREF: sub_402600+398↓r
.text:00402944 58                            pop     eax
.text:00402945 89 85 C4 FD FF FF             mov     [ebp+var_23C], eax
.text:0040294B E8 03 00 00 00                call    loc_402953
.text:0040294B
.text:0040294B                               ; ---------------------------------------------------------------------------
.text:00402950 EA                            db 0EAh
.text:00402951                               ; ---------------------------------------------------------------------------
.text:00402951 EB 09                         jmp     short loc_40295C
.text:00402951
.text:00402953                               ; ---------------------------------------------------------------------------
.text:00402953
.text:00402953                               loc_402953:                             ; CODE XREF: sub_402600+34B↑j
.text:00402953 5B                            pop     ebx
.text:00402954 43                            inc     ebx
.text:00402955 53                            push    ebx
.text:00402956 B8 11 11 11 11                mov     eax, 11111111h
.text:0040295B C3                            retn
.text:0040295C                               ; ---------------------------------------------------------------------------
.text:0040295C
.text:0040295C                               loc_40295C:                             ; CODE XREF: sub_402600+351↑j
.text:0040295C E8 07 00 00 00                call    loc_402968
.text:0040295C
.text:00402961 BB 33 33 33 33                mov     ebx, 33333333h
.text:00402966 EB 0D                         jmp     short loc_402975
.text:00402966
.text:00402968                               ; ---------------------------------------------------------------------------
.text:00402968
.text:00402968                               loc_402968:                             ; CODE XREF: sub_402600:loc_40295C↑p
.text:00402968 BB 11 11 11 11                mov     ebx, 11111111h
.text:0040296D 5B                            pop     ebx
.text:0040296E BB 75 29 40 00                mov     ebx, offset loc_402975
.text:00402973 53                            push    ebx
.text:00402974 C3                            retn
```

结果这些垃圾指令没有搞出很复杂的分支结构，都是顺着跳到下一段，只需要全部`nop`掉就行了。

2AF0, 2CA0, 2e80这几个函数同理。去花之后发现他们的作用分别是去掉flag和括号并反向，rc4初始化，rc4加密。下面有个循环，对比密文是否正确。

密文是2600开头那一长段赋值（`5bD6D026C8DD197E6E3ECB16917DFFAFDD7664B0F7E58957829F0C009ED045FA`），密码是`qwertyuiop`

丢到CyberChef就出结果了。记得反向。记得包flag{}。

---

PS：

开始的时候完全不敢全部NOP掉，因为有对eax和ebx赋值，害怕他们很重要。结果留下了一堆JMP。ida: 6。后来看了大佬的去花脚本，发现是直接nop掉了。。。那个脚本是python2的，为了跑起来，我给改成python3的了。

```python
from ida_bytes import get_bytes, patch_bytes
import re
addr = 0x402600
end = 0x402ae3
buf = get_bytes(addr, end-addr)
def handler1(s):
    s = s.group(0)
    print("".join(["%02x"%i for i in s]))
    s = b"\x90"*len(s)
    return s
p = b"\xe8\x00\x00\x00\x00.*?\xc3.*?\xc3"
buf = re.sub(p, handler1, buf, flags=re.I)
patch_bytes(addr, buf)
print("Done")
```

不过这个匹配好像有点问题，rc4加密那个函数会被多nop掉很多，所以上面写的范围是仅限sub_402600。（如果已经猜到是rc4加密了的话就没什么影响了）。

---

原以为MFC我还是知道的，结果一头雾水，现在才知道有xspy这个东西。
