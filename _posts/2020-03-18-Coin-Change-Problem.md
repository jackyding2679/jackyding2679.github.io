---
layout: post
title: '经典算法-凑硬币问题'
date: 2020-03-18
author: Jacky
categories: 经典算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 回溯算法 动态规划
---
https://testversions-1302272194.cos.ap-guangzhou.myqcloud.com/unnamed.jpg 

### 提出问题
假设我们有几种不同币值的硬币 v1，v2，……，vn（单位是元）。
如果我们要支付 w 元，求最少需要多少个硬币。比如，我们有 3 种不同的硬币，
1 元、3 元、5 元，我们要支付 9 元，最少需要 3 个硬币（3 个 3 元的硬币）。

我们把问题具体化，现在有1、3、5、6、7等五种硬币，需要支付31元。
该问题可以有很多种解法，我们从易到难一个个分析。

### 暴力求解
通过回溯算法，穷举所有可能，直到找到最小值，这种算法的复杂度最高，但
比较容易实现，先看代码。
```clike
//回溯算法，暴力求解，穷举所有可能
//g_min_coin_num存储最小值
//调用方法GetMinCoinNum1(31, 0)
int GetMinCoinNum1(int price, int coin_num) {
    if (price <= 0) {
        if (price == 0 && coin_num < g_min_coin_num) {
            g_min_coin_num = coin_num;
        }
        return 0;
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            GetMinCoinNum1(price - g_coins[i], coin_num + 1);
        }
    }
}
```
上述代码也可以写成
```clike
	if (price - 1 >= 0) {
        GetMinCoinNum1(price - 1, coin_num + 1);
    }
	if (price - 3 >= 0) {
        GetMinCoinNum1(price - 3, coin_num + 1);
    }
	if (price - 5 >= 0) {
        GetMinCoinNum1(price - 5, coin_num + 1);
    }
	if (price - 6 >= 0) {
        GetMinCoinNum1(price - 6, coin_num + 1);
    }
	if (price - 7 >= 0) {
        GetMinCoinNum1(price - 7, coin_num + 1);
    }
```

以1，3，5硬币支付9元为例，暴力穷举算法会计算所有可能，所以会有大量的重复
子问题，f(9,0)表示需要支付9元，当前已经选择的硬币数量为0，如下为整个
穷举过程中的递归树：
![](/assets/img/Change-Coin-Problem.jpg)
图中相同颜色的节点表示相同的子问题，所以我们可以进行优化。

### 暴力求解优化
对于已知的价格，比如f(5,x)，表示需要支付5元，当前已经选择了x个硬币，
这种情况，我们知道我们有5元的硬币，可以直接选择5元的硬币，而不用
去计算所有可能的情况，比如1+1+1+1+1,1+1+3,3+1+1等等，对于跟已知的硬币相同
的价格，可以直接返回1个硬币。优化方法如下：
```clike
//回溯算法，暴力求解，部分优化
//状态转移方程
//f(price) = min(f(price - 1), f(price - 3), f(price - 5), f(price - 6), f(price - 7))
int GetMinCoinNum2(int price) {
    int num[COIN_NUM];
    //部分优化，遇到硬币直接返回1
    for (int i = 0; i < COIN_NUM; i++) {
        if (price == g_coins[i]) {
            return 1;
        }
    }

    if (price == 0) {
        return 0;
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            //硬币+1，减去硬币值
            num[i] = 1 + GetMinCoinNum2(price - g_coins[i]);
        } else {
            //本次的硬币组合不能凑成price
            num[i] = INT_MAX;
        }
    }
    return  Min(num, COIN_NUM);
}
```
通过这个优化，可以减少一些计算量，但其实如果价格很高，而硬币数量比较少的时候，
还是需要大量的递归运算，比如价格为31，那计算可能会计算很多次f(15,x)，那下面
的优化方法可以解决该问题。

### 动态规划+备忘录
上面提到可能会计算很多次f(15,x),那假如第一次计算完f(15，x)后，我把f(15)的值
保存下来，下一次需要计算f(15,x)，就可以直接查表得到值，而不用再计算。
```clike
//初始化备忘录
g_mem[0] = 0;
g_mem[1] = 1;
g_mem[3] = 1;
g_mem[5] = 1;
g_mem[6] = 1;
g_mem[7] = 1;

//动态规划+备忘录
//状态转移方程
//f(price) = min(f(price - 1), f(price - 3), f(price - 5), f(price - 6), f(price - 7))
int GetMinCoinNum3(int price) {
    int num[COIN_NUM];
    if (price == 0) {
        return 0;
    }
    //遇到备忘录，直接返回
    if (g_memo[price] != INT_MAX) {
        return g_memo[price];
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            num[i] = 1 + GetMinCoinNum3(price - g_coins[i]);
        } else {
            num[i] = INT_MAX;
        }
    }
    int tmp = Min(num, COIN_NUM);
    //记录备忘录，记录的是最小值
    g_mem[price] = tmp;
    return tmp;
}
```
这样，除了基础的1，3，5等硬币，可以直接返回1，其他的价格，比如15，也可以直接查表返回结果
，当然第一次记录f(15)的时候，记录的是f(15)的最小值，下次可以直接使用。
相比上面的方法，这次优化可以使性能有很大的提升，当然，还有更快的方案。

### 终极方案：动态规划+递推公式
```clike
//动态规划+备忘录+递推公式
//mem[i] = min(mem[i - 1] , mem[i - 3], mem[i - 5], mem[i - 6], mem[i - 7])
int GetMinCoinNum4(int price) {
    int num[COIN_NUM];
    //初始化备忘录
    g_mem[0] = 0;
    g_mem[1] = 1;
    g_mem[2] = 2;
    g_mem[3] = 1;
    g_mem[4] = 2;
    g_mem[5] = 1;
    g_mem[6] = 1;
    g_mem[7] = 1;
    //从基础硬币的下一个开始
    for (int i = g_coins[COIN_NUM] + 1; i <= g_price; i++) { //从8开始
        for (int j = 0; j < COIN_NUM; j++) {
            num[i] =  g_mem[i - g_coins[j]];//min(g_mem[8 - 1], gmem[8 - 3], gmem[8 - 5], g[mem - 6], g[mem - 7])
        }
        g_mem[i] = 1 + Min(num, COIN_NUM);
    }
    return g_mem[price];
}
```
这种方案，首先初始化备忘录，对于已知的硬币，直接返回1，对于2元或者4元这种价格，我们
也很容易知道最少硬币数量为2。
如果需要计算的价格是7以内，我们直接返回结果。如果超过7元，比如8元，我们有多种方案，
要么f(7)+1个1元硬币，或者f(5)+1个3元硬币，或者f(3)+1个5元硬币，或者f(2)+1个6元硬币，或者f(1)+1个7元硬币，
有这么多种方法，我们取以上五种方法的最小值，就是支付8元需要的最小硬币数量。我们把这个值存下来，
同理，我们计算f(9)，f(10)...f(31)。这种方法就是动态规划的递推公式，速度非常快。
	
### 四种方案比较
全部代码如下
```clike
//硬币找零问题，假设我们有几种不同币值的硬币 v1，v2，……，vn（单位是元）。
//如果我们要支付 w 元，求最少需要多少个硬币。比如，我们有 3 种不同的硬币，
//1 元、3 元、5 元，我们要支付 9 元，最少需要 3 个硬币（3 个 3 元的硬币）
//Author: jackyding
//Date: 2020/03/15
//Description:
//使用回溯算法以及动态规划
#include<iostream>
#include<climits>
#include<string.h>
#include<sys/time.h>

#define COIN_NUM 5
int g_coins[COIN_NUM] = {1,3,5,6,7};

int g_price = 31;

int g_min_coin_num = INT_MAX;

int *g_memo;

//获取最小值
int Min(int *p, int size) {
    int min = INT_MAX;
    for (int i = 0; i < size; ++i) {
        if (p[i] < min) {
            min = p[i];
        }
    }
    return min;
}

//回溯算法，暴力求解，穷举所有可能
//g_min_coin_num存储最小值
int GetMinCoinNum1(int price, int coin_num) {
    if (price <= 0) {
        if (price == 0 && coin_num < g_min_coin_num) {
            g_min_coin_num = coin_num;
        }
        return 0;
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            GetMinCoinNum1(price - g_coins[i], coin_num + 1);
        }
    }
}

//回溯算法，暴力求解，部分优化
//状态转移方程
//f(price) = min(f(price - 1), f(price - 3), f(price - 5), f(price - 6), f(price - 7))
int GetMinCoinNum2(int price) {
    int num[COIN_NUM];
    //部分优化，遇到硬币直接返回1
    for (int i = 0; i < COIN_NUM; i++) {
        if (price == g_coins[i]) {
            return 1;
        }
    }

    if (price == 0) {
        return 0;
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            //硬币+1，减去硬币值
            num[i] = 1 + GetMinCoinNum2(price - g_coins[i]);
        } else {
            //本次的硬币组合不能凑成price
            num[i] = INT_MAX;
        }
    }
    return  Min(num, COIN_NUM);
}

//动态规划+备忘录
//状态转移方程
//f(price) = min(f(price - 1), f(price - 3), f(price - 5), f(price - 6), f(price - 7))
int GetMinCoinNum3(int price) {
    int num[COIN_NUM];
    if (price == 0) {
        return 0;
    }
    //遇到备忘录，直接返回
    if (g_memo[price] != INT_MAX) {
        return g_memo[price];
    }
    for (int i = 0; i < COIN_NUM; i++) {
        if (price - g_coins[i] >= 0) {
            num[i] = 1 + GetMinCoinNum3(price - g_coins[i]);
        } else {
            num[i] = INT_MAX;
        }
    }
    int tmp = Min(num, COIN_NUM);
    //记录备忘录
    g_mem[price] = tmp;
    return tmp;
}

//动态规划+备忘录+递推公式
//mem[i] = min(mem[i - 1] , mem[i - 3], mem[i - 5], mem[i - 6], mem[i - 7])
int GetMinCoinNum4(int price) {
    int num[COIN_NUM];
    //初始化备忘录
    g_mem[0] = 0;
    g_mem[1] = 1;
    g_mem[2] = 2;
    g_mem[3] = 1;
    g_mem[4] = 2;
    g_mem[5] = 1;
    g_mem[6] = 1;
    g_mem[7] = 1;
    //从基础硬币的下一个开始
    for (int i = g_coins[COIN_NUM] + 1; i <= g_price; i++) { //从8开始
        for (int j = 0; j < COIN_NUM; j++) {
            num[i] =  g_mem[i - g_coins[j]];//min(g_mem[8 - 1], gmem[8 - 3], gmem[8 - 5], g[mem - 6], g[mem - 7])
        }
        g_mem[i] = 1 + Min(num, COIN_NUM);
    }
    return g_mem[price];
}

int main() {
    int time_use;
    struct timeval start;
    struct timeval end;

    g_mem = new int [g_price + 1];
    for (int i = 0; i < g_price + 1; i++) {
        g_mem[i] = INT_MAX;
    }
    g_mem[0] = 0;
    g_mem[1] = 1;
    g_mem[3] = 1;
    g_mem[5] = 1;
    g_mem[6] = 1;
    g_mem[7] = 1;

    gettimeofday(&start,NULL);
    GetMinCoinNum1(g_price, 0);
    gettimeofday(&end,NULL);
    time_use = (end.tv_sec-start.tv_sec)*1000000+(end.tv_usec-start.tv_usec);
    std::cout << "backtracking algm 1, g_min_coin_num=" << g_min_coin_num << ", time_use=" << time_use << "us" << std::endl;

    gettimeofday(&start,NULL);
    int tmp = GetMinCoinNum2(g_price);
    gettimeofday(&end,NULL);
    time_use = (end.tv_sec-start.tv_sec)*1000000+(end.tv_usec-start.tv_usec);
    std::cout << "backtracking algm with optimizing, g_min_coin_num=" << tmp << ", time_use=" << time_use << "us" << std::endl;

    gettimeofday(&start,NULL);
    tmp = GetMinCoinNum3(g_price);
    gettimeofday(&end,NULL);
    time_use = (end.tv_sec-start.tv_sec)*1000000+(end.tv_usec-start.tv_usec);
    std::cout << "dymanic programing algm with memo, g_min_coin_num=" << tmp << ", time_use=" << time_use << "us" << std::endl;

    gettimeofday(&start,NULL);
    tmp = GetMinCoinNum4(g_price);
    gettimeofday(&end,NULL);
    time_use = (end.tv_sec-start.tv_sec)*1000000+(end.tv_usec-start.tv_usec);
    std::cout << "dymanic programing algm with memo and recursion formula, g_min_coin_num=" << tmp << ", time_use=" << time_use << "us" << std::endl;

    delete [] g_mem;
    return 0;
}
```
如下比较四种方案的耗时：
```clike
# ./Change-Coin-Probelm
backtracking algm 1, g_min_coin_num=5, time_use=69145us
backtracking algm 2, g_min_coin_num=5, time_use=8200us
dymanic programing algm with memo, g_min_coin_num=5, time_use=2us
dymanic programing algm with memo and recursion formula, g_min_coin_num=5, time_use=0us
```
可见四种方案耗时差距明显，第一种方案耗时69145us，第二种耗时8200us，第三种耗时2us，最后的方案耗时为0us。
