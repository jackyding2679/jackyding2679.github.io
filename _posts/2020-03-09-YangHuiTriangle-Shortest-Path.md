---
layout: post
title: '算法经典-杨辉三角最短路径'
date: 2020-03-09
author: Jacky
categories: 经典算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 动态规划算法 杨辉三角最短路径问题
---

使用动态规划算解决杨辉三角最短路径问题。

### 提出问题
改造杨辉三角，让每层的数据变成随机的整数，即数据不再满足
原本杨辉三角的规律，编程求出从最高层移动到最底层的最短路径长度，并
给出最短的路径。

原始的杨辉三角如下图：
![](/assets/img/Original_YangHuiTriangle.jpg)

改造后的杨辉三角如下图所示：
![](/assets/img/Special_YangHuiTriangle.jpg)

这个图形有点像二叉树，但是这个图中，部分节点有两个父节点，图中的箭头表示路径
中下一步的方向，需要计算出从顶层到最底层的最短路径。

用动态规划（Dynamic Programing）解决该问题的核心思路是记录到达每一层时的所有状态，
比如从顶层到第二层，一共会有两种状态:5+7->12和5+6->11，把这两种状态记录下来，那接着
走到第三层，本来会有4种状态:2+12->14，3+12->15，3+11->14，4+11->15，如果每一层都把
所有可能的情况都记录下来，那没到后面一层，状态会越来越多，所以此处可以有一个优化，
即第二层的7和6走到子节点3的时候，可以选择比较小的那个，即6节点当前路径的长度为11，7节点当前路径的
长度为12，所以此时选择6节点，那第三层只有三种状态：2+12->14，3+11->14，4+11->15：
![](/assets/img/Special_YangHuiTriangle1.jpg)

按照这种算法，每一层的状态其实就与每个节点个数相同。

另外，处理算法的时候，需要注意每层的第一个节点和最后一个节点只有一个父节点，而其他节点
有两个父节点，子节点和父节点的关系是：levelN[i]的父节点为levelN-1[i1]和LevelN-1[i]。

当计算完最后一层的状态后，可以通过比较得到最短路径，以及最短路径中最后一层的节点，
接着可以往上层计算每一层的节点。

可见，动态规划算法明显比回溯算法的算法复杂度低。


### 动态规划算法的实现
```clike
//杨辉三角最短路径问题 
//改造杨辉三角，让每层的数据变成随机的整数，即数据不再满足
//原本杨辉三角的规律，编程求出从最高层移动到最底层的最短路径长度，并
//给出最短的路径。
//Author: jackyding
//Date: 2020/03/07
//Description: Find shortest path for YangHuiTriangle
//Algorithm: Dynamic Porogramming
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

#define TRIANGLE_LEVEL 6
char data[TRIANGLE_LEVEL][TRIANGLE_LEVEL] =
{
    {5},
    {7, 6},
    {2, 3, 4},
    {4, 5, 6, 1},
    {6, 2, 3, 4, 7},
    {2, 1, 3, 5, 4, 4}
};
int level_stat[TRIANGLE_LEVEL][TRIANGLE_LEVEL];
int shortest_path[TRIANGLE_LEVEL];
int shortest_len;

void PrintStat() {
    printf("level stat is:\n");
    for (int i = 0; i < TRIANGLE_LEVEL; i++) {
        for (int j = 0; j < TRIANGLE_LEVEL; j++) {
            printf("%2d ", level_stat[i][j]);
        }
        printf("\n");
    }
    printf("---------------------------------\n");
}

void PrintTri(bool mark = false) {
    int row_size, start_print_off, printed_num;
    bool should_print, finish_row;
    int column = 2 * TRIANGLE_LEVEL - 1;

    if (mark) {
        printf("shortest path is:\n");
    } else {
        printf("origin triangle is:\n");
    }
    for (int i = 0; i < TRIANGLE_LEVEL; i++) {
        row_size = i + 1;
        start_print_off = TRIANGLE_LEVEL - 1 - i;
        printed_num = 0;
        should_print = true;
        finish_row = false;
        for (int j = 0; j < column; j++) {
            if (j < start_print_off || finish_row) {
                printf("   ");
                continue;
            }

            if (should_print) {
                if (mark && shortest_path[i] == printed_num) {
                    printf("[%d]", data[i][printed_num++]);
                } else {
                    printf(" %d ", data[i][printed_num++]);
                }
                should_print = false;
                if (printed_num == row_size) {
                    finish_row = true;
                }
            } else {
                should_print = true;
                printf("   ");
            }
        }
        printf("\n");
     }
    printf("---------------------------------\n");
}

int GetMin(int a, int b) {
    return a < b ? a : b;
}

void GenLevelStat(int level)  {
    if (level == 0) {
        level_stat[level][0] = data[0][0];
        return;
    }
    for (int i = 0; i <= level; i++) {
        if (i == 0) {
            //本层第一个
            level_stat[level][i] = data[level][i] + level_stat[level - 1][0];
        } else if (i == level) {
            //本层最后一个
            level_stat[level][i] = data[level][i] + level_stat[level - 1][level - 1];
        } else {
            level_stat[level][i] = data[level][i] + GetMin(level_stat[level - 1][i - 1], level_stat[level - 1][i]);
        }
    }
}

void GetShortestPath() {
    int k = 0;
    shortest_len = level_stat[TRIANGLE_LEVEL - 1][0];
    for (int i = 1; i < TRIANGLE_LEVEL - 1; i++) {
        if (shortest_len > level_stat[TRIANGLE_LEVEL - 1][i]) {
            shortest_len = level_stat[TRIANGLE_LEVEL - 1][i];
            k = i;
        }
    }
    printf("shortest len is %d\n", shortest_len);
    int last_shortest_len;
    for (int level = TRIANGLE_LEVEL - 1; level >= 0; level--) {
        shortest_path[level] = k;
        if (level == 0) {
            break;
        }
        if (k == 0) {
            //k = k
        } else if (k == level) {
            k = k - 1;
        } else {
            last_shortest_len = level_stat[level][k] - data[level][k];
            if (level_stat[level - 1][k - 1] == last_shortest_len) {
                k = k - 1;
            } else if (level_stat[level - 1][k] == last_shortest_len) {
                //k = k;
            } else {
                assert(0);
            }
       }
    }
    printf("---------------------------------\n");
}
int main() {
    PrintTri();
    for (int i = 0; i < TRIANGLE_LEVEL; i++) {
        GenLevelStat(i);
    }
    PrintStat();
    GetShortestPath();
    PrintTri(true);
    return 0;
}

```
### 输出结果
```clike
# ./YangHuiTriangle
origin triangle is:
                5
             7     6
          2     3     4
       4     5     6     1
    6     2     3     4     7
 2     1     3     5     4     4
---------------------------------
level stat is:
 5  0  0  0  0  0
12 11  0  0  0  0
14 14 15  0  0  0
18 19 20 16  0  0
24 20 22 20 23  0
26 21 23 25 24 27
---------------------------------
shortest len is 21
---------------------------------
shortest path is:
               [5]
            [7]    6
         [2]    3     4
      [4]    5     6     1
    6    [2]    3     4     7
 2    [1]    3     5     4     4
---------------------------------
```