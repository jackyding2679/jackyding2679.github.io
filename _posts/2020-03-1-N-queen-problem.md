---
layout: post
title: '算法经典-八皇后问题'
date: 2020-02-29
author: Jacky
categories: 经典算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 回溯算法 八皇后问题
---

回溯的处理思想，有点类似枚举搜索。我们枚举所有的解，
找到满足期望的解。为了有规律地枚举所有可能的解，避免遗漏和重复，
我们把问题求解的过程分为多个阶段。每个阶段，我们都会面对一个岔路口，
我们先随意选一条路走，当发现这条路走不通的时候（不符合期望的解），
就回退到上一个岔路口，另选一种走法继续走。

### 提出问题
我们有一个 8x8 的棋盘，希望往里放 8 个棋子（皇后），
每个棋子所在的行、列、对角线都不能有另一个棋子。
八皇后问题就是期望找到所有满足这种要求的放棋子方式。

例如，我们以一个4*4的棋盘来举例，#代表一个棋子，
如下棋子放置方式不满足要求:
```clike
* # * *
* * * #
# * * *
* # * *
```
4*4棋盘有两种放置方法，如下:
```css
* # * *
* * * #
# * * *
* * # *
```
```css
* * # *
# * * * 
* * * #
* # * *
```

### 算法实现
用C代码实现该算法，核心思想是，针对每一行，判断每一个棋子是否满足要求，
如果不满足要求，则判断下一列，
```clike
//八皇后问题，有个8*8的棋盘，需要在每一行放置一个棋子，
//每个棋子横，竖，斜着方向都没有其他棋子，要求输出结果
//Author: jackyding
//Date: 2020/02/28
//Description: solve 8-queen-probelm
#include "stdio.h"
#include "stdlib.h"

const int g_chessboard_size = 8;
int *g_result = new int [g_chessboard_size];

//判断该位置是否可以放置棋子
bool IsPointOk(int row, int column) {
    int left_column = column - 1, right_column = column + 1;
    for (int i = row - 1; i >= 0; i--) {
        if (g_result[i] == column) {
            return false;
        }
        if (left_column >= 0) {
            if (left_column == g_result[i]) {
                return false;
            }
            left_column--;
        }
        if (right_column < g_chessboard_size) {
            if (right_column == g_result[i]) {
              return false;
            }
            right_column++;
        }
    }
    return true;
}

void PrintResult() {
    for (int i = 0; i < g_chessboard_size; i++) {
        for (int j = 0; j < g_chessboard_size; j++) {
            if (g_result[i] == j) {
                printf("# ");
            } else {
                printf("* ");
            }
        }
        printf("\n");
    }
    printf("- - - - - - - - \n");
}

//核心代码，检查每一行
void CheckRow(int row) {
    if (row == g_chessboard_size) {
        PrintResult();
        return;
    }

    for (int column = 0; column < g_chessboard_size; column++) {
        if (IsPointOk(row, column)) {
            g_result[row] = column;
            CheckRow(row + 1);
        }
    }
}

int main() {
    CheckRow(0);
    return 0;
}
```

### 输出结果
```clike
# ./N-Queens-Problem
# * * * * * * *
* * * * # * * *
* * * * * * * #
* * * * * # * *
* * # * * * * *
* * * * * * # *
* # * * * * * *
* * * # * * * *
- - - - - - - -
# * * * * * * *
* * * * * # * *
* * * * * * * #
* * # * * * * *
* * * * * * # *
* * * # * * * *
* # * * * * * *
* * * * # * * *
- - - - - - - -
# * * * * * * *
* * * * * * # *
* * * # * * * *
* * * * * # * *
* * * * * * * #
* # * * * * * *
* * * * # * * *
* * # * * * * *
- - - - - - - -
...
还有很多结果
```