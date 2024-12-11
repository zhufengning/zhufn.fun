---
title: 某道Android ctf题的wp
urlname: 某道Android ctf题的wp
date: 2024-12-11 19:18:42
tags:
  - Android
---
今日[Android小技巧](https://zhufn.fun/archives/AndroidRe/)：
> 不要用jadx和jd，反编译出的控制结构可能是错的，用jeb

~~第一次出Android题喵，出得烂不要骂我喵~~
# Java部分

谷歌最近推荐用Compsoe取代传统xml布局，我一看，赶紧写一个试试，AI生成完一看，跟个flutter似的。丢进反编译一看，跟个答辩似的，请看对比图（这两张图大致在同一个地方）（不开混淆的话可能会好很多）：
![](assets/2024-12-11%20-%20某道Android-ctf题的wp/file-20241211203752899.png)
![](assets/2024-12-11%20-%20某道Android-ctf题的wp/file-20241211203820280.png)
为什么我能认出来这两个地方在一起呢，这就要从我们的解题思路说起了。
根据题目描述（虽然是最后几个小时加的但相信大家自己看也能看出来），这道题考的是JNI，再看最顶上的几个方法，上面都标着native，肯定就是了：
```java
static {
System.loadLibrary("getmyvip");
}

public final native String CheckVip(String arg1) 
public final native String Greetings(String arg1)
public final native String Hello(String arg1)
public final native String World(String arg1)
```
通过交叉引用能进到这个回调函数（对应的类）
```java
public final class e implements a {
    public final MainActivity d;
    public final e0 e;
    public final e0 f;
    public final e0 g;
    public final e0 h;
    public final e0 i;

    public e(MainActivity mainActivity0, e0 e00, e0 e01, e0 e02, e0 e03, e0 e04) {
        this.d = mainActivity0;
        this.e = e00;
        this.f = e01;
        this.g = e02;
        this.h = e03;
        this.i = e04;
    }

    @Override  // u1.a
    public final Object d() {
        MainActivity mainActivity0 = this.d;
        i.e(mainActivity0, "this$0");
        e0 e00 = this.e;
        e0 e01 = this.f;
        i.e(e01, "$dialogMessage$delegate");
        e0 e02 = this.g;
        e0 e03 = this.h;
        i.e(e03, "$showDialog$delegate");
        e0 e04 = this.i;
        e01.setValue(mainActivity0.Hello(((String)e00.getValue())));
        if(g.S(((String)e01.getValue()), "VIP")) {
            e02.setValue("VIP Activated");
        }
        else if(g.S(((String)e01.getValue()), "114514")) {
            e02.setValue(mainActivity0.Greetings(((String)e00.getValue())) + mainActivity0.World(((String)e00.getValue())) + mainActivity0.CheckVip(((String)e00.getValue())));
        }
        else {
            e02.setValue("Not VIP");
        }

        Boolean boolean0 = Boolean.TRUE;
        e03.setValue(boolean0);
        e04.setValue(boolean0);
        return l.a;
    }
}
```
是不是跟原本代码里的onClick一模一样。为什么回调函数会生成一个类呢？答案是为了捕获外层变量（闭包嘛）（从构造函数传引用进去）。再来看看逻辑部分，要进入VIP Activated分支，必须要使Hello函数的返回值里包含VIP。而114514那个分支则是出题人乱写的，只要我们打开so看一看`Greetings`, `World`, `CheckVip`这几个函数就能马上意识到（除非你认为出题人的代码在程序运行后会patch自己，但稍加验证就知道并不会）。

# 原生部分

打开so（可以直接用jeb打开并反编译，这样我们搞Android逆向就不用多买一份IDA了喵），查找`Greetings`, `World`, `CheckVip`这3个函数，发现他们都长得一样
```cpp
jstring Java_fun_1_zhufn_getmyvip_MainActivity_Greetings(JNIEnv* param0, jobject param1, jstring param2) {
    return (jstring)NewStringUTF((long)param0, "Hello from JNI !", (long)param2);
}
```
但是`Hello`函数却找不到。众所周知JNI函数有两种加载方式，一种是按照格式命名自动加载，另一种是通过api提交映射表。所以我们打开so的JNI入口点：`JNI_OnLoad`函数。（对了我前面说不用IDA是开玩笑的，arm的反编译器一般7和9的盗版都有带）
```cpp
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  jint v3; // [xsp+Ch] [xbp-24h]
  void *v5[2]; // [xsp+20h] [xbp-10h] BYREF

  v5[1] = *(void **)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  word_5777C = 3328;
  sub_2436C(5LL, &unk_5777E);
  sub_2436C(38LL, &unk_57784);
  sub_2436C(32LL, &unk_577AB);
  v5[0] = 0LL;
  v3 = -1;
  if ( !(unsigned int)_JavaVM::GetEnv((_JavaVM *)vm, v5, 65540) && (sub_24AE0(v5[0]) & 0x80000000) == 0 )
    v3 = 65540;
  _ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));
  return v3;
}
```
首先开局的三个`sub_2436C`非常可疑，他们会进行一些神秘的异或，并且每次会对传入的参数（word_5777C）进行神秘的修改
```cpp
__int64 __fastcall sub_2436C(__int64 result, __int64 a2)
{
  int i; // [xsp+Ch] [xbp-14h]
  int v4; // [xsp+1Ch] [xbp-4h]

  v4 = result;
  for ( i = 0; i < v4; ++i )
  {
    *(_BYTE *)(a2 + i) ^= word_5777C;
    result = sub_246C8(&word_5777C);
  }
  return result;
}

_WORD *__fastcall sub_246C8(_WORD *result)
{
  *result = (((unsigned __int16)(((unsigned __int16)*result << 15 >> 15) ^ ((unsigned __int16)*result << 13 >> 15) ^ ((unsigned __int16)*result << 12 >> 15)) ^ ((unsigned __int16)*result << 10 >> 15)) << 15)
          + ((int)(unsigned __int16)*result >> 1);
  return result;
}
```
提取数据，模拟这个过程，能够得到三个字符串，分别是函数名，签名，类名，这是注册一个JNI函数的必要信息。之前在群里发的跟ai聊天的[链接](https://poe.com/s/UlucXXaER15OsAKf4PdO)完成的就是到这一步。
```
unk_5777E: Hello
unk_57784: (Ljava/lang/String;)Ljava/lang/String;
byte_577AB: fun_/zhufn/getmyvip/MainActivity
```
```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>


// sub_246C8 函数
uint16_t *__fastcall sub_246C8(uint16_t *result)
{
  *result = (((uint16_t)(((uint16_t)*result << 15 >> 15) ^ ((uint16_t)*result << 13 >> 15) ^ ((uint16_t)*result << 12 >> 15)) ^ ((uint16_t)*result << 10 >> 15)) << 15)
          + ((int)(uint16_t)*result >> 1);
  return result;
}

// sub_2436C 函数
void sub_2436C(size_t length, uint8_t *data, uint16_t *key) {
    for (size_t i = 0; i < length; ++i) {
        data[i] ^= (*key & 0xFF); // XOR 解密
        sub_246C8(key);           // 更新 key
    }
}

// 打印解密结果
void print_decrypted(const char *label, const uint8_t *data, size_t length) {
    printf("%s: ", label);
    for (size_t i = 0; i < length; ++i) {
        printf("%c", data[i]);
    }
    printf("\n");
}

int main() {
    // 初始化加密数据
    uint8_t unk_5777E[] = { 0x48, 0xE5, 0x2C, 0xCC, 0xBF };
    uint8_t unk_57784[] = { 0x40, 0x78, 0x70, 0x6C, 0x70, 0x62, 0x2E, 0xEC, 0x21, 0x4E, 0x77, 0xA7, 0x97, 0x96, 0x83, 0x91,
                            0x12, 0x59, 0xA4, 0x66, 0xEB, 0xB9, 0x88, 0x02, 0x5B, 0xB2, 0xA2, 0x86, 0x9D, 0x9E, 0xD3, 0xAD,
                            0x0B, 0x4D, 0xF6, 0x21, 0xC0, 0x68 };
    uint8_t byte_577AB[] = { 0xCF, 0xA1, 0x84, 0x2A, 0x15, 0xE7, 0xA6, 0x92, 0x95, 0x97, 0xD3, 0x19, 0xDA, 0x2B, 0xC2, 0x2E,
                             0xDD, 0x3C, 0xDA, 0xFA, 0xA7, 0x14, 0xD3, 0x33, 0x6F, 0xF4, 0x3F, 0x4C, 0x64, 0x60, 0xF0, 0x3B,
                             0x00, 0x00, 0x00, 0x00 };

    // 初始化解密密钥
    uint16_t word_5777C = 3328;

    // 解密数据
    sub_2436C(5, unk_5777E, &word_5777C);
    sub_2436C(38, unk_57784, &word_5777C);
    sub_2436C(32, byte_577AB, &word_5777C);

    // 打印解密结果
    print_decrypted("unk_5777E", unk_5777E, sizeof(unk_5777E));
    print_decrypted("unk_57784", unk_57784, sizeof(unk_57784));
    print_decrypted("byte_577AB", byte_577AB, sizeof(byte_577AB));

    return 0;
}

```
下面是JNINativeMethod结构体的定义，你会发现里面没有class，这是因为我们在注册JNI函数时，需要拿到class对象，而不是提供一个类名字符串
```c
typedef struct {
    const char* name;
    const char* signature;
    void*       fnPtr;
} JNINativeMethod;
```
本题中你可以直接在data段里找到这个结构体
```
.data:00000000000577D0 off_577D0       DCQ unk_5777E           ; DATA XREF: sub_24AE0+44↑o
.data:00000000000577D8                 DCQ unk_57784
.data:00000000000577E0                 DCQ _Z8__JNIFunP7_JNIEnvP8_jobjectP8_jstring ; __JNIFun(_JNIEnv *,_jobject *,_jstring *)
```
也可以在这里找到`RegisterNatives`调用
```cpp
bool __fastcall sub_24AE0(_JNIEnv *a1)
{
  __int64 Class; // [xsp+8h] [xbp-18h]

  Class = _JNIEnv::FindClass(a1, byte_577AB);
  return Class && (int)_JNIEnv::RegisterNatives(a1, Class, &off_577D0, 1LL) >= 0;
}
```
你可能发现了，其实根本不需要解密那几个字符串，但先不管这一点，打开`Hello`函数本体，你可能会嫌c++代码反编译出来就是依托，但相信在我进行了变量重命名后你肯定就懂了。
```cpp
__int64 __fastcall __JNIFun(_JNIEnv *a1, __int64 a2, __int64 a3)
{
  __int64 v3; // x0
  char v5; // [xsp+24h] [xbp-12Ch]
  int i; // [xsp+68h] [xbp-E8h]
  __int64 v8; // [xsp+90h] [xbp-C0h]
  _BYTE vector_correct_result[24]; // [xsp+98h] [xbp-B8h] BYREF
  _QWORD vector_result[3]; // [xsp+B0h] [xbp-A0h] BYREF
  _BYTE str_input[24]; // [xsp+C8h] [xbp-88h] BYREF
  char v12[16]; // [xsp+E0h] [xbp-70h] BYREF
  char v13[16]; // [xsp+F0h] [xbp-60h] BYREF
  __int128 v14; // [xsp+100h] [xbp-50h] BYREF
  _OWORD v15[2]; // [xsp+110h] [xbp-40h]
  __int128 v16; // [xsp+130h] [xbp-20h] BYREF
  char v17; // [xsp+140h] [xbp-10h]
  __int64 v18; // [xsp+148h] [xbp-8h]

  v18 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  if ( first_time )
  {
    first_time = 0;
    _JNIEnv::GetStringUTFChars(a1, a3, 0LL);
    std::string::basic_string[abi:ne180000]<0>();
    v3 = str_size((__int64)str_input);
    vector_from_str(vector_result, v3);
    for ( i = 0; i < (unsigned __int64)vector_size(vector_result); ++i )
    {
      v5 = *(_BYTE *)str_sub((__int64)str_input, i) ^ xor_global;
      *(_BYTE *)vector_sub(vector_result, i) = v5;
      next_value(&xor_global);
    }
    *(_OWORD *)((char *)v15 + 11) = *(__int128 *)((char *)&xmmword_1673E + 11);
    v15[0] = xmmword_1673E;
    v14 = xmmword_1672E;
    vector_from_initializer_list(vector_correct_result, &v14, 43LL);
    if ( (vector_operator_equal(vector_result, vector_correct_result) & 1) != 0 )
    {
      xor_global = init_value_2;
      *(_QWORD *)&v13[6] = 0xCA87E6099E519ELL;
      *(_QWORD *)v13 = 0x519E9A81BED712B8LL;
      do_xor(13LL, (__int64)v13);
      v8 = _JNIEnv::NewStringUTF(a1, v13);
    }
    else
    {
      xor_global = init_value_2;
      *(_QWORD *)&v12[5] = 0xF68E7EDB199494LL;
      *(_QWORD *)v12 = 0x199494CEB8D016A9LL;
      do_xor(12LL, (__int64)v12);
      v8 = _JNIEnv::NewStringUTF(a1, v12);
    }
    vector_destruct(vector_correct_result);
    vector_destruct(vector_result);
    std::string::~string(str_input);
  }
  else
  {
    xor_global = init_value_2;
    v17 = 0;
    v16 = xmmword_153E9;
    do_xor(16LL, (__int64)&v16);
    v8 = _JNIEnv::NewStringUTF(a1, (const char *)&v16);
  }
  _ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));
  return v8;
}
```
## 如何看出一个c++函数的作用
顺便介绍一下如何看出一个c++函数的作用，以`vector_operator_equal`为例，我们点开它：
```cpp
__int64 __fastcall vector_operator_equal(__int64 a1, __int64 a2)
{
  __int64 v2; // x0
  __int64 v4; // [xsp+0h] [xbp-40h]
  char v5; // [xsp+Ch] [xbp-34h]
  __int64 v6; // [xsp+18h] [xbp-28h]
  __int64 v7; // [xsp+20h] [xbp-20h]

  v4 = vector_size(a1);
  v5 = 0;
  if ( v4 == vector_size(a2) )
  {
    v7 = sub_26670(a1);
    v6 = sub_266A0(a1);
    v2 = sub_26670(a2);
    v5 = sub_265E8(v7, v6, v2);
  }
  return v5 & 1;
}
```
看似有很多函数，但大多数点进最深处都是这3种形式：
```cpp
//成员访问
__int64 __fastcall vector_size(_QWORD *a1)
{
  return a1[1] - *a1;
}

//赋值
_QWORD *__fastcall sub_26A20(_QWORD *result, __int64 a2)
{
  *result = a2;
  return result;
}

//解引用
__int64 __fastcall sub_269A4(__int64 a1)
{
  return *(_QWORD *)a1;
}
```
最后在return之前的有效操作只有这个memcmp，因此我们就确定了这个c++函数的作用
```cpp
bool __fastcall sub_26820(const void *a1, const void *a2, size_t a3)
{
  return memcmp(a1, a2, a3) == 0;
}
```
# 最后
解密__JNIFun的主要分支里的字符串，能找到返回vip的那个分支（其实可以直接猜正确结果肯定是两个vector相等）
再把xmmword_1672E那里的数据拿出来，放在ai写的那个解密后面（因为这里接着用了解密函数名时用的异或数）异或一遍就好了。
本题源码（校内访问）：<https://driver.neu-nex.fun/%E6%AF%94%E8%B5%9B%E5%BD%92%E6%A1%A3/2024/%E7%BB%83%E4%B9%A0%E8%B5%9B%E6%96%87%E4%BB%B6/GetMyVip.zip>
