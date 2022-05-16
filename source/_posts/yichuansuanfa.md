---
title: 遗传算法实现旅行商问题
date: 2022-05-16 12:29:00
categories: 数据科学
urlname: yichuansuanfa
tags:
---
我们选择遗传算法的经典案例——旅行商问题来介绍遗传算法的具体实现。

##  旅行商问题

给定一系列城市和每对城市之间的距离，求解访问每一座城市一次并回到起始城市的最短回路。

我们将给每个城市设定一个坐标，以此来求得每对城市之间的距离。对于图上问题，可使用Floyd算法对图进行处理，以获得每对城市之间的最短路。

## 全局常量、变量定义
```cpp
const int SIZE = 1000, CNT = 1000;//种群大小和最大代数
using Pos = pair<double, double>;//坐标类型
using Rst = vector<int>;//染色体类型
vector<Pos> a;//城市坐标数组
double ans = 1e9;//全局最优解，初始化为一个不可能达到的大的长度
Rst ans_rst;//该解对应的染色体
double notans = 1e9;//当前种群最优解，（所以不是答案）
double geneSum = 0;//当前种群的参与二元锦标赛的所有解的和
int geneCnt = 0;//当前种群参与二元锦标赛的个体数量，好像不会变
//以上两行用来求种群平均解，用来观察种群变化
```

## 工具函数

```cpp
//取随机浮点数，0~1
double randf() {
    return static_cast<float>(rand()) / static_cast<float>(RAND_MAX);
}
//两点距离
double getDis(Pos p1, Pos p2) {
    return sqrt(pow(p1.first - p2.first, 2) + pow(p1.second - p2.second, 2));
}
```

## 主要过程
```cpp
int main() {
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        double tx, ty;
        scanf("%lf%lf", &tx, &ty);
        a.push_back(make_pair(tx, ty));
    }
    //以上为数据输入部分
    //整个种群
    vector<Rst> pool; 
    //生成随机染色体
    for (int i = 1; i <= SIZE; ++i)
        pool.push_back(randGene(n));
    //进行CNT代繁衍
    for (int i = 1; i <= CNT; ++i) {
        //初始化本轮统计数据
        notans = 1e9;
        geneSum = 0;
        geneCnt = 0;
        printf("%d ", i);
        //适应度数组
        vector<double> score;
        //新的种群
        vector<Rst> new_pool;
        for (int i = 1; i <= CNT / 2; ++i) {
            //选择两个亲代
            auto win1 = choose(pool);
            auto win2 = choose(pool);
            //20%的概率进行交叉
            if (randf() < 0.2) {
                auto children = oxCross(win1, win2);
                win1 = children.first;
                win2 = children.second;
            }
            //尝试变异，1%的概率变异
            bianYi(win1);
            bianYi(win2);
            //插入新的种群
            new_pool.push_back(win1);
            new_pool.push_back(win2);
        }
        //输出本轮结果
        printf("%lf %lf %lf\n", ans, notans, geneSum / geneCnt);
        //种群改变
        pool = new_pool;
    }
    //输出最优解的染色体
    for (int v : ans_rst)
        printf("%d ", v);
    return 0;
}
```

## 基因编码与产生

我们以旅行商走过的顺序作为基因的编码，染色体是长度为城市数量的10进制序列。
```cpp
//随机生成染色体
Rst randGene(int n) {
    Rst ret;
    unordered_map<int, bool> mp;
    for (int i = 1; i <= n; ++i) {
        int newr = rand() % n;
        while (mp[newr])
            newr = rand() % n;
        ret.push_back(newr);
        mp[newr] = true;
    }
return ret;
}
```

## 适应度评价

取总路程的倒数作为适应度。该方法产生的适应度总和不为1.
```cpp
//走的总路程
double getValue(Rst &g) {
    int len = g.size();
    double s = 0;
    for (int i = 1; i < len; ++i)
        s += getDis(a[g[i - 1]], a[g[i]]);
    s += getDis(a[g[0]], a[g[len - 1]]);
    if (s < ans) {
        ans = s;
        ans_rst = g;
    }
    if (s < notans)
        notans = s;
    geneSum += s;
    geneCnt++;
    return s;
}

//适应度
double getShiYingDu(Rst &g) { return 1.0 / getValue(g); }

```

##  选择

我们采用二元锦标赛的方式进行选择，即每次随机抽出两个个体，保留适应度更高的。相应地，选择多个个体进行比较的方法即为n元锦标赛。

```cpp
//选择，二元锦标赛
Rst &choose(vector<Rst> &pool) {
    int len = pool.size();
    int i = rand() % len;
    int j = rand() % len;
    int big = getShiYingDu(pool[i]) > getShiYingDu(pool[j]) ? i : j;
    return pool[big];
}

```

## 交叉

我们使用的交叉方式是一种顺序交叉。从个体2中随机选取一段放在个体1前面，并将其之后的重复基因去掉。再使用另一个个体中的同一段做同样的操作。

例如1 5 4 3 2和5 3 2 1 4随机选择第1到3位得到的子代是2 1 3 5 4和5 4 3 2 1

```cpp
//顺序交叉(Order Crossover)
pair<Rst, Rst> oxCross(Rst &r1, Rst &r2) {
    int len = r1.size();
    int i = rand() % len, j = i + rand() % (len - i);
    Rst s1, s2;
    unordered_map<int, bool> mp1, mp2;
    for (int p = i; p <= j; ++p) {
        s1.push_back(r2[p]);
        mp1[r2[p]] = 1;
        s2.push_back(r1[p]);
        mp2[r1[p]] = 1;
    }
    for (int p = 0; p < len; ++p) {
        if (!mp1[r1[p]]) {
            s1.push_back(r1[p]);
            mp1[r1[p]] = 1;
        }
        if (!mp2[r2[p]]) {
            s2.push_back(r2[p]);
            mp2[r2[p]] = 1;
        }
    }
    return {s1, s2};
}
```
## 变异

一定概率随机选两个基因进行交换。

```cpp
//变异
void bianYi(Rst &r) {
    double rd = randf();
    if (rd < 0.01) {
        int len = r.size();
        int i = rand() % len;
        int j = rand() % len;
        int t = r[i];
        r[i] = r[j];
        r[j] = t;
    }
}
```

 

##  运行测试

我们随机生成了一组测试数据，包含174个城市坐标。下图是进行了1000次繁衍后的最优解（红）和种群平均解（绿）。

![img](http://pic.yupoo.com/zhufn/66c71941/86413658.png)

## 完整代码
```cpp
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <ctime>
#include <iostream>
#include <unordered_map>
#include <utility>
#include <vector>
using namespace std;

const int SIZE = 1000, CNT = 1000;
using Pos = pair<double, double>;
using Rst = vector<int>;

vector<Pos> a;
double ans = 1e9;
double notans = 1e9;
double geneSum = 0;
int geneCnt = 0;
Rst ans_rst;

//取随机浮点数，0~1
double randf() {
    return static_cast<float>(rand()) / static_cast<float>(RAND_MAX);
}

//随机生成解
Rst randGene(int n) {
    Rst ret;
    unordered_map<int, bool> mp;
    for (int i = 1; i <= n; ++i) {
        int newr = rand() % n;
        while (mp[newr])
            newr = rand() % n;
        ret.push_back(newr);
        mp[newr] = true;
    }
    return ret;
}

//两点距离
double getDis(Pos p1, Pos p2) {
    return sqrt(pow(p1.first - p2.first, 2) + pow(p1.second - p2.second, 2));
}

//走的总路程
double getValue(Rst &g) {
    int len = g.size();
    double s = 0;
    for (int i = 1; i < len; ++i) {
        s += getDis(a[g[i - 1]], a[g[i]]);
    }
    s += getDis(a[g[0]], a[g[len - 1]]);
    if (s < ans) {
        ans = s;
        ans_rst = g;
    }

    if (s < notans)
        notans = s;

    geneSum += s;
    geneCnt++;
    return s;
}

//适应度
double getShiYingDu(Rst &g) { return 1.0 / getValue(g); }

//选择，二元锦标赛
Rst &choose(vector<Rst> &pool) {
    int len = pool.size();
    int i = rand() % len;
    int j = rand() % len;
    int big = getShiYingDu(pool[i]) > getShiYingDu(pool[j]) ? i : j;
    return pool[big];
}

//顺序交叉(Order Crossover)
pair<Rst, Rst> oxCross(Rst &r1, Rst &r2) {
    int len = r1.size();
    int i = rand() % len, j = i + rand() % (len - i);
    Rst s1, s2;
    unordered_map<int, bool> mp1, mp2;
    for (int p = i; p <= j; ++p) {
        s1.push_back(r2[p]);
        mp1[r2[p]] = 1;
        s2.push_back(r1[p]);
        mp2[r1[p]] = 1;
    }
    for (int p = 0; p < len; ++p) {
        if (!mp1[r1[p]]) {
            s1.push_back(r1[p]);
            mp1[r1[p]] = 1;
        }
        if (!mp2[r2[p]]) {
            s2.push_back(r2[p]);
            mp2[r2[p]] = 1;
        }
    }
    return {s1, s2};
}

//变异
void bianYi(Rst &r) {
    double rd = randf();
    if (rd < 0.01) {
        int len = r.size();
        int i = rand() % len;
        int j = rand() % len;
        int t = r[i];
        r[i] = r[j];
        r[j] = t;
    }
}

int main() {
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        double tx, ty;
        scanf("%lf%lf", &tx, &ty);
        a.push_back(make_pair(tx, ty));
    }
    //以上为数据输入部分
    //整个种群
    vector<Rst> pool; 
    //生成随机染色体
    for (int i = 1; i <= SIZE; ++i)
        pool.push_back(randGene(n));
    //进行CNT代繁衍
    for (int i = 1; i <= CNT; ++i) {
        //初始化本轮统计数据
        notans = 1e9;
        geneSum = 0;
        geneCnt = 0;
        printf("%d ", i);
        //适应度数组
        vector<double> score;
        //新的种群
        vector<Rst> new_pool;
        for (int i = 1; i <= CNT / 2; ++i) {
            //选择两个亲代
            auto win1 = choose(pool);
            auto win2 = choose(pool);
            //20%的概率进行交叉
            if (randf() < 0.2) {
                auto children = oxCross(win1, win2);
                win1 = children.first;
                win2 = children.second;
            }
            //尝试变异，1%的概率变异
            bianYi(win1);
            bianYi(win2);
            //插入新的种群
            new_pool.push_back(win1);
            new_pool.push_back(win2);
        }
        //输出本轮结果
        printf("%lf %lf %lf\n", ans, notans, geneSum / geneCnt);
        //种群改变
        pool = new_pool;
    }
    //输出最优解的染色体
    for (int v : ans_rst)
        printf("%d ", v);
    return 0;
}

```
