---
title: '[xctf.reverse]notsequence（用杨辉三角当密码）'
date: 2022-08-02 18:46:00
categories: reverse
urlname: reverse_yanghui
tags:
---
有这么一道题，它检验输入，并把输入里的空白删掉后取md5作为flag。它的输入非常神奇，用scanf读入一堆整数。放进数组里后要经过两个函数的检验。  
（下面请ghidra发言）  
```c
int check1(int *nums)
{
  int j;
  int s;
  int i;
  int res;
  
  res = 0;
  i = 0;
  while( true ) {
     if (0x400 < i) {
       return res;
     }
     if (nums[i] == 0) break;
     s = 0;
     for (j = 0; j <= res; j = j + 1) {
       s = s + nums[i + j];
     }
     if (1 << ((byte)res & 0x1f) != s) {
       return -1;
     }
     i = ((res + 2) * (res + 1)) / 2;
     res = res + 1;
  }
  return res;
}

//param_20传入的是check1的返回值（杨辉三角形层数，main里检测了它是否等于20）
int check2(int *nums,int param_20)
{
  int j;
  int s;
  int i;
  int i0;
  
  i0 = 0;
  i = 1;
  while( true ) {
    if (para_20 <= i) {
      return 1;
    }
    s = 0;
    j = i + -1;
    if (nums[i] == 0) break;
    for (; j < para_20 + -1; j = j + 1) {
      s = s + nums[i0 + ((j + 1) * j) / 2];
    }
    if (nums[i + ((j + 1) * j) / 2] != s) {
      return 0;
    }
    i0 = i0 + 1;
    i = i + 1;
  }
  return 0;
}
```
看writeup后：  
check1中，res是层数，i = (res + 2) * (res + 1) / 2，是每层开始的下标。check1是在检验第res层的和是不是i的res次方。（成功被res&0x1f搞蒙了。。效果是取res末五位，可能是防越界？好像不是。汇编里是SHL EBX,CL  ，EBX里是1，太怪了）
check2检测的是“斜线上数字的和等于其向左（从左上方到右下方的斜线）或向右拐弯（从右上方到左下方的斜线），拐角上的数字。”意会一下。
所以最后提交的是RCTF{20层去皮杨辉三角形的md5}