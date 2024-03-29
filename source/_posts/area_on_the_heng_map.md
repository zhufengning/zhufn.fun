---
title: 横截面图上的区域
date: 2021-11-05 20:48
tags: [栈]
urlname: hengjiemiantu

---
你的任务是模拟洪水灾害。  
对于给定的横截面图，给出淹没部分的面积。  
[![横截面图](https://judgeapi.u-aizu.ac.jp/resources/images/IMAGE2_ALDS1_3_D)](https://judgeapi.u-aizu.ac.jp/resources/images/IMAGE2_ALDS1_3_D)  
假设该地区不断地下雨，而从该地区溢出的水正在落入两侧的海中。 例如，对于上面的横截面图，雨水将产生洪水，其面积分别为 4、2、1、19 和 9。  
**输入**  
在一行中给出一个字符串，分别用"/",""和"_"表示斜坡和平原。 例如，上面例子的区域由字符串“\ \ / / / \ _ / \ / \ \ \ \ / _ / \ \ / / / _ _ \ \ \ _ \ \ / _ \ / _ / ”给出。  
**输出**  
按以下格式输出：  
A  
K  L1,L2,…,Lk  
在第一行，打印出洪水区域的总面积A。  
在第二行，打印出洪水区域的个数K和每个区域的面积Li（从左到右）。  
**约束**  
1 ≤ length of the string  ≤ 20000  

看到这道题，我首先想到我们可以从头开始，一格一格向后找到开始往下凹的地方，持续跟踪直到高度与之前持平。然后继续往后直到又开始下降然后重复之前的过程。。  
然后我发现，很容易就会遇到，再也达不到之前开始下降的高度 的情况，于是我果断地放弃这种方案，写了一个大模拟：  
直接构建一个横截面图，用0代表空气，1代表山体，2代表整格水，3代表下降的半格水，4代表上升的半格水  
先把山以上的部位全部填满水，然后重复执行：将左右下方任一方向是空气的水扔掉，直到再也扔不掉为止，剩下的就是我们的积水  
最后用dfs找到每一坨水，算出面积  
然后我们发现地图是个二维数组，定义成`ditu[MAXN][MAXN]`的话会直接爆空间。然后我们考虑到应该不会有一直下降或上升的山所以高度用不到那么多格，于是尝试开小第二维。  
喜提两个TLE  
```c
#include <stdio.h>

#define MAXN 20005
#define INF ((int)0x3f3f3f3f)
#define min(a, b) ((a) > (b) ? (b) : (a))
#define max(a, b) ((a) < (b) ? (b) : (a))
int a[MAXN] = {0};
int ditu[MAXN][1005] = {0};//0:empty,1:shan,2:shui,3:downshan,4:upshan
short vis[MAXN][1005] = {0};
int mmjimf[MAXN] = {0};
int newm = 0;

double dfs(int x, int y)
{
    if (vis[x][y])
    {
        return 0.;
    }
    vis[x][y] = 1;

    double value = 0;
    switch (ditu[x][y])
    {
        case 2:
            value = 1;
            break;
        case 3:
        case 4:
            value = 0.5;
            break;
        default:
            return 0;
    }
    if (ditu[x][y] == 2) value += dfs(x + 1, y) + dfs(x - 1, y) + dfs(x, y + 1) + dfs(x, y - 1);
    else if (ditu[x][y] == 3)
    {
        value += dfs(x + 1, y)  + dfs(x, y + 1) + dfs(x, y - 1);
        if (ditu[x - 1][y] != 4)
        {
            value += dfs(x - 1, y);
        }
    } else if (ditu[x][y] == 4)
    {
        value += dfs(x - 1, y)  + dfs(x, y + 1) + dfs(x, y - 1);
        if (ditu[x + 1][y] != 3)
        {
            value += dfs(x + 1, y);
        }
    }
    return value;
}

int main(void)
{
    char c = getchar();
    int newp = 1;
    int zvdi = INF, zvgc = -INF;
    while (c == '/' || c == '\\' || c == '_')
    {
        ++newp;
        switch (c)
        {
            case '/':
                a[newp] = a[newp - 1] + 1;
                break;
            case '\\':
                a[newp] = a[newp - 1] - 1;
                break;
            case '_':
                a[newp] = a[newp - 1];
                break;
        }
        if (a[newp] < zvdi)
        {
            zvdi = a[newp];
        }

        c = getchar();
    }
    for (int i = 1; i <= newp; ++i)
    {
        a[i] += (-zvdi + 1);
        if (a[i] > zvgc)
        {
            zvgc = a[i];
        }
    }

    for (int i = 1; i < newp; ++i)
    {
        int low = min(a[i], a[i + 1]);
        int high = max(a[i], a[i + 1]);
        for (int j = 1; j <= zvgc; ++j)
        {
            if (j <= low)
            {
                ditu[i][j] = 1;
            } else if (j >= low && j <= high)
            {
                if (low != high)
                {
                    if (a[i] > a[i + 1]) ditu[i][j] = 3;
                    else ditu[i][j] = 4;
                } else
                {
                    ditu[i][j] = 2;
                }

            } else if (j > high)
            {
                ditu[i][j] = 2;
            }
        }
    }
    int vcdc = 0;
    do
    {
        vcdc = 0;
        for (int j = zvgc; j >= 1; --j)
        {
            for (int i = 1; i < newp; ++i)
            {
                if (ditu[i][j] == 2 || ditu[i][j] == 3 || ditu[i][j] == 4)
                {
                    if (!((ditu[i - 1][j] || ditu[i][j] == 3) && (ditu[i + 1][j] || ditu[i][j] == 4) && ditu[i][j - 1]))
                    {
                        ++vcdc;
                        ditu[i][j] = 0;
                    }
                }
            }
        }
    } while (vcdc);
    double mmji = 0;
    for (int j = zvgc; j >= 1; --j)
    {
        for (int i = 1; i < newp; ++i)
        {
            switch (ditu[i][j])
            {
                case 2:
                    mmji += 1;
                    break;
                case 3:
                case 4:
                    mmji += 0.5;
                    break;
            }
        }
    }
    printf("%.0lf\n", mmji);
    for (int i = 1; i < newp; ++i)
    {
        for (int j = zvgc; j >= 1; --j)
        {
            if (!vis[i][j])
            {
                double ans = dfs(i, j);
                if (ans) mmjimf[++newm] = ans;
            }
        }
    }

    printf("%d ", newm);
    for (int i = 1; i <= newm; ++i)
    {
        printf("%d ", mmjimf[i]);
    }
    return 0;
}
```

遇到严重超时TLE，我内心十分悲痛，打开了必应，搜索全网，搜到了一个题解。这个题解告诉我们  
> 由淹没区域可以想到 ‘\’ 与 '/' 的配对。  
> 此题比较特别的是如何组成一整个被水淹没的区间？答案是将所有配对的区间中 相互包含的区间合并。  
> 如何合并呢？可以知道大区间所包含的小区间必定比大区间先入栈，则可以依此合并。  
> 如何计算横截面积？所有配对的‘\’与'/'之间形成的横截面都为梯形或三角形，把问题分解为多个小问题来解答。  

我看完大受启发，决定按此思路写一写。写了一段后发现，这哪里用得到栈，好像只需要加加减减就可以了。  
这时我发现，只要我把最开始的做法变得更暴力一点，从每一个起点出发，如果再也达不到之前开始下降的高度，就扔掉这个起点，换下一个起点。这样做不仅不会浪费太多空间，而且比大模拟快很多，顺利AC。  
```c
#include <stdio.h>

#define MAXN 20005

char sgn[MAXN] = {0};
double ans[MAXN] = {0};
double tot = 0;
int news = 0;
int newa = 0;

int main()
{
    int h = 0;
    char t = getchar();
    while (t == '/' || t == '\\' || t == '_')
    {
        sgn[++news] = t;
        t = getchar();
    }
    int i = 1;
    while (i <= news)
    {
        double sum = .5, hi = 1;
        if (sgn[i] != '\\')
        {
            ++i;
            continue;
        }
        int st = 1;
        int j;
        for (j = i + 1; j <= news && st > 0; ++j)
        {
            if (sgn[j] == '\\')
            {
                ++st;
                ++hi;
                sum += (hi - .5);
            } else if (sgn[j] == '/')
            {
                --st;
                --hi;
                sum += (hi + .5);
            } else
            {
                sum += hi;
            }
        }
        if (st == 0)
        {
            i = j;
            ans[++newa] = sum;
            tot += sum;
        } else
        {
            ++i;
        }
    }

    printf("%.0lf\n%d ", tot, newa);
    for (int i = 1; i <= newa; ++i)
    {
        printf("%.0lf ", ans[i]);
    }
    return 0;
}
```
复健之路进行中~~