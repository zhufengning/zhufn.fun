---
title: "[2019_Redhat]childRE"
urlname: "[2019_Redhat]childRE"
date: January 3, 2023 4:05 PM
tags: [BFS, DFS, reverse, 二叉树]
---
64位PE无壳。打开main函数。

题目来源：xctf

首先判断输入长度是不是31位。不是的话就卡死。

```c
scanf("%s",input);
lVar5 = -1;
do {
 str_len = lVar5 + 1;
 lVar4 = lVar5 + 1;
 lVar5 = str_len;
} while (input[lVar4] != '\0');
if (str_len != 31) {
 do {
   Sleep(1000);
 } while( true );
}
```

根据其他师傅的wp，下面要从最后的字符对比开始分析。

```c
while( true ) {
   if ("1234567890-=!@#$%^&*()_+qwertyuiop[]QWERTYUIOP{}asdfghjkl;\'ASDFGHJKL:\"ZXCVBNM<>?zxcvbnm, ./"
      [(int)(&out_DAT_1400056c0)[lVar5] % 0x17] != (&DAT_140003478)[lVar5]) {
                 /* WARNING: Subroutine does not return */
     _exit(_Code);
   }
   if ("1234567890-=!@#$%^&*()_+qwertyuiop[]QWERTYUIOP{}asdfghjkl;\'ASDFGHJKL:\"ZXCVBNM<>?zxcvbnm, ./"
      [(int)(&out_DAT_1400056c0)[lVar5] / 0x17] !=
      "55565653255552225565565555243466334653663544426565555525555222"[lVar5]) break;
   _Code = _Code + 1;
   lVar5 = lVar5 + 1;
   if (0x3d < _Code) {
     printf("flag{MD5(your input)}\n");
     __security_check_cookie(uVar3 ^ (ulonglong)auStack_58);
     return extraout_EAX_00;
   }
 }
```

把`DAT_140003478`的值复制出来写脚本。

```python
_Code = 0
lVar5 = 0
out_DAT_1400056c0 = [" "] * 0x3e
DAT_140003478 = [ 0x28, 0x5f, 0x40, 0x34, 0x36, 0x32, 0x30, 0x21, 0x30, 0x38, 0x21, 0x36, 0x5f, 0x30, 0x2a, 0x30, 0x34, 0x34, 0x32, 0x21, 0x40, 0x31, 0x38, 0x36, 0x25, 0x25, 0x30, 0x40, 0x33, 0x3d, 0x36, 0x36, 0x21, 0x21, 0x39, 0x37, 0x34, 0x2a, 0x33, 0x32, 0x33, 0x34, 0x3d, 0x26, 0x30, 0x5e, 0x33, 0x26, 0x31, 0x40, 0x3d, 0x26, 0x30, 0x39, 0x30, 0x38, 0x21, 0x36, 0x5f, 0x30, 0x2a, 0x26 ]
while True:
    ans = 0
    print(DAT_140003478[lVar5])
    for i in range(128):
        print(i, end=" ")
        if ord("""1234567890-=!@#$%^&*()_+qwertyuiop[]QWERTYUIOP{}asdfghjkl;\'ASDFGHJKL:\"ZXCVBNM<>?zxcvbnm,. /"""[i % 0x17]) != DAT_140003478[lVar5]:              
            continue
        if ord("""1234567890-=!@#$%^&*()_+qwertyuiop[]QWERTYUIOP{}asdfghjkl;\'ASDFGHJKL:\"ZXCVBNM<>?zxcvbnm,. /"""[i // 0x17]) !=         ord("55565653255552225565565555243466334653663544426565555525555222"[lVar5]):
            continue
        ans = i
        break
    out_DAT_1400056c0[_Code] = ans
    _Code = _Code + 1;
    lVar5 = lVar5 + 1;
    print()
    if (0x3d < _Code):
        print("".join(map(chr,out_DAT_1400056c0)))
        break
```

得到`private: char * __thiscall R0Pxx::My_Aut0_PWN(unsigned char *)`

再往前：

```c
UnDecorateSymbolName(symbol_name,&out_DAT_1400056c0,0x100,0);
lVar5 = -1;
do {
 lVar4 = lVar5 + 1;
 pcVar1 = &DAT_1400056c1 + lVar5;
 lVar5 = lVar4;
} while (*pcVar1 != '\0');
if (lVar4 != 62) {
 this = FUN_1400018a0((longlong *)cout_exref);
 std::basic_ostream<char,struct_std::char_traits<char>_>::operator<<
          ((basic_ostream<char,struct_std::char_traits<char>_> *)this,FUN_140001a60);
 __security_check_cookie(uVar3 ^ (ulonglong)auStack_58);
 return extraout_EAX;
}
```

关于UnDecorateSymbolName，众所周知编译器会把符号弄成人看不懂的形式（说起来以前见过一个叫mangle的词）（千万不要用ecosia搜mangled这个词，会出血腥图，呕呕呕）（啊，这个是gcc和clang的东西，见[http://web.mit.edu/tibbetts/Public/inside-c/www/mangling.html](http://web.mit.edu/tibbetts/Public/inside-c/www/mangling.html)）

扯远了，vs的规则可以在[https://zh.wikipedia.org/wiki/Visual_C%2B%2B名字修饰](https://zh.wikipedia.org/wiki/Visual_C%2B%2B%E5%90%8D%E5%AD%97%E4%BF%AE%E9%A5%B0) 查看。但是更方便的方法是直接把这个函数写出来，然后用`__FUNCDNAME__`宏输出。

```cpp
#include <cstdio>

class R0Pxx {
public:
	R0Pxx() { My_Aut0_PWN(nullptr); }
private: 
	char* __thiscall My_Aut0_PWN(unsigned char* a) {
		printf(__FUNCDNAME__); 
		return nullptr; 
	}
};

int main() {
	auto a = R0Pxx();
}
```

得到`?My_Aut0_PWN@R0Pxx@@AAEPADPAE@Z`

另外插一句嘴，一定要用能分得清0和O的字体。cmd的字体可以设为点阵字体（不知道为什么vs的输出窗口字体只有几种能选的，如果设成别的就会找不到字体然后掉回宋体）。

最后我们剩下的就是这一段了：

```cpp
iVar2 = FUN_140001280(input);
root = (Node *)CONCAT44(extraout_var,iVar2);
symbol_name = &sym_name_DAT_1400057c0;
if (root != (Node *)0x0) {
 dfs(root->left);
 dfs(root->right);
 symbol_name[i_DAT_1400057e0] = root->v;
 i_DAT_1400057e0 = i_DAT_1400057e0 + 1;
}
```

省流：动态调试可以发现是一个位置上的置换，按它的方式换回去就可以了。（但我还是分析了一下这个二叉树）

`Node*`类型是我们定义的结构体（为什么是‘们’呢因为ghidra的自动生成帮了一部分忙）。

| Offset | Length | Type | Name |
| --- | --- | --- | --- |
| 0 | 1 | char | v |
| 1~7 | 1 | undefined |  |
| 8 | 8 | Node * | left |
| 16 | 8 | Node * | right |

然后是dfs函数，在把它的名字改成dfs之前，我还看不懂它。

不过现在很明显了，是一个后序遍历，存到`sym_name`里面

```cpp
void dfs(Node *param_1)
{
 if (param_1 != (Node *)0x0) {
   dfs(param_1->left);
   dfs(param_1->right);
   (&sym_name_DAT_1400057c0)[i_DAT_1400057e0] = param_1->v;
   i_DAT_1400057e0 = i_DAT_1400057e0 + 1;
 }
 return;
}
```

用ida调试查看一下root的结构，看看树建成了什么样：

（ida可能无法识别把一个内存中的变量分到两个寄存器这个操作，比如这里需要在v6上右键，映射到v4。）

![Untitled](/pics/%5B2019_Redhat%5DchildRE%208add79fc8f66434ea6fbff78a3be12f0/Untitled.png)

这个叫什么二叉树来着我给忘了，反正就是按层的顺序，从左到右。

顺便练一下用f#写数据结构。

先深搜把数据填进去，然后广搜还原建树的顺序。

```fsharp
open System.Collections.Generic

type Node =
    { v: char
      l: Node option
      r: Node option }

let mutable cnt = 0

let s = "?My_Aut0_PWN@R0Pxx@@AAEPADPAE@Z"

let rec dfs (dep: int) (p: Node) : Node =
    if dep <= 5 then
        let r =
            { p with
                l = Some({ v = ' '; l = None; r = None } |> dfs (dep + 1))
                r = Some({ v = ' '; l = None; r = None } |> dfs (dep + 1))
                v = s[cnt] }

        cnt <- cnt + 1
        r
    else
        p

let root = dfs 1 { v = ' '; l = None; r = None }

let q = new Queue<Node>()

q.Enqueue root

while q.Count > 0 do
    let t = q.Dequeue()
    printf "%c" t.v

    if t.l.IsSome then
        q.Enqueue t.l.Value

    if t.r.IsSome then
        q.Enqueue t.r.Value
```

得到：`Z0@tRAEyuP@xAAA?M_A0_WNPx@@EPDP`

最后md5一下就好了。

碎碎念：用尝试f#写二叉树的时候我最开始想把结点搞成可变的。结果就变成了`type Node ={ v: char;l: Node ref option;r: Node ref option}`

想取到里面的`Node`？先解两层包。然后引用传参最开始写的是Node ref类型，写完给我个蓝色warning，然后才知道现在都是写byref。然后返回的时候还不能直接取成员地址，让我先let绑定。

然后我就崩溃了，一查才知道原来是要修改结点的时候直接扔掉返回一个新的。啊，我忘了该用函数式写法了。（然而我还是用了一个全局变量计数。）

其实这道题是我随机到的，就是感觉按难度挨着做，中间突然冒出几道新题，无法满足强迫症的感觉，干脆从这里打破吧。然后随机到这个难度6的题。然后点进建树的函数被吓出来。然后开摆看wp。然后告诉我这是签到题。呜呜呜。
