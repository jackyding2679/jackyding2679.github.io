---
layout: post
title: '算法经典-数独问题'
date: 2020-02-29
author: Jacky
categories: 算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 回溯算法 数独
---

> No content

### 提出问题
现在有一个9*9大小的表格，表格中有一些有已知的数据，推理出所有剩余空格的数字，
并满足每一行、每一列、每一个同色九宫内的数字均含1-9，且不重复。

如下一个9*9的表格，里面已经填了一些数组
```
0, 0, 5, 3, 0, 0, 0, 0, 0
8, 0, 0, 0, 0, 0, 0, 2, 0
0, 7, 0, 0, 1, 0, 5, 0, 0
4, 0, 0, 0, 0, 5, 3, 0, 0
0, 1, 0, 0, 7, 0, 0, 0, 6
0, 0, 3, 2, 0, 0, 0, 8, 0
0, 6, 0, 5, 0, 0, 0, 0, 9
0, 0, 4, 0, 0, 0, 0, 3, 0
0, 0, 0, 0, 0, 9, 7, 0, 0
```
可以使用回溯算法实现，基本思想是，填入一个满足条件的数字，再填入下一列，如果到了一行的末尾
则切换到下一行，如果某一个位置所有的数子都不满足要求，则回到上一个位置，尝试另外一个数字。

如果想增加难度，可以要求找到所有满足条件的结果。

### 算法实现（C）
```
//Auther:jackyding
//Date:2020/02/28
//Description:find soduku
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#define SUDU_SIZE 9
int array[SUDU_SIZE][SUDU_SIZE] = {
 {0, 0, 5, 3, 0, 0, 0, 0, 0},
 {8, 0, 0, 0, 0, 0, 0, 2, 0},
 {0, 7, 0, 0, 1, 0, 5, 0, 0},
 {4, 0, 0, 0, 0, 5, 3, 0, 0},
 {0, 1, 0, 0, 7, 0, 0, 0, 6},
 {0, 0, 3, 2, 0, 0, 0, 8, 0},
 {0, 6, 0, 5, 0, 0, 0, 0, 9},
 {0, 0, 4, 0, 0, 0, 0, 3, 0},
 {0, 0, 0, 0, 0, 9, 7, 0, 0}
 };
bool get_result = false;

void PrintResult() {
    int i, j;
    for (i = 0; i < SUDU_SIZE; i++) {
        for (j = 0; j < SUDU_SIZE; j++) {
            printf("%d ", array[i][j]);
        }
        printf("\n");
    }
    printf("-------------------\n");
}

//判断填入的数字是否满足要求
bool IsOk(int row, int column, int num) {
    int i;
    for (i = 0; i < SUDU_SIZE; i++) {
        if (i != column && array[row][i] == num) {
            return false;
        }
        if (i != row && array[i][column] == num) {
            return false;
        }
    }
    return true;
}

//判断最后的结果是否满足数独的要求
bool IsSoduku() {
    int row, column, i;
    for (row = 0; row < SUDU_SIZE; row++) {
        for (column = 0; column < SUDU_SIZE; column++) {
            if (!IsOk(row, column, array[row][column])) {
                return false;
            }
        }
    }
    return true;
}

void Sudoku(int row, int column) {
    int i, j;
    if (get_result) return;
    if (row == SUDU_SIZE && column == 0) {
        if (IsSoduku()) {
            printf("Yeah!!!find a Soduku\n");
            PrintResult();
            get_result = true;
        }
        return;
    }

    if (array[row][column] == 0) {
        for (i = 1; i <= SUDU_SIZE; i++) {
            if (IsOk(row, column, i)) {
                array[row][column] = i;
                column++;
				//切换至下一行
                if (column == SUDU_SIZE) {
                    row++;
                    column = 0;
                }
                Sudoku(row, column);
                column--;
                if (column < 0) {
                    row--;
                    column = SUDU_SIZE - 1;
                }
                array[row][column] = 0;
                if (get_result) {
					//找到其他结果
                    if (row == 0 && column == 0) {
                        get_result = false;
                        printf("search another Soduku\n");
                        continue;
                    }
                }
            }
        }
    } else {
        column++;
        if (column == SUDU_SIZE) {
             row++;
             column = 0;
        }
        Sudoku(row, column);
    }
    return;
}

int main() {
    printf("origin data:\n");
    PrintResult();
    Sudoku(0,0);
    return 0;
}
```

### 输出结果
```
[root@node1 backtracking]# ./Sudoku-Problem
origin data:
0 0 5 3 0 0 0 0 0
8 0 0 0 0 0 0 2 0
0 7 0 0 1 0 5 0 0
4 0 0 0 0 5 3 0 0
0 1 0 0 7 0 0 0 6
0 0 3 2 0 0 0 8 0
0 6 0 5 0 0 0 0 9
0 0 4 0 0 0 0 3 0
0 0 0 0 0 9 7 0 0
-------------------
Yeah!!!find a Soduku
1 2 5 3 4 6 8 9 7
8 3 1 4 6 7 9 2 5
2 7 6 9 1 8 5 4 3
4 8 9 6 2 5 3 7 1
9 1 2 8 7 3 4 5 6
7 5 3 2 9 1 6 8 4
3 6 7 5 8 4 2 1 9
6 9 4 7 5 2 1 3 8
5 4 8 1 3 9 7 6 2
-------------------
search another Soduku
Yeah!!!find a Soduku
2 4 5 3 6 1 8 9 7
8 3 1 6 4 7 9 2 5
6 7 2 9 1 3 5 4 8
4 8 7 1 9 5 3 6 2
3 1 9 8 7 2 4 5 6
7 9 3 2 5 6 1 8 4
1 6 8 5 3 4 2 7 9
9 5 4 7 2 8 6 3 1
5 2 6 4 8 9 7 1 3
-------------------
search another Soduku
Yeah!!!find a Soduku
6 2 5 3 4 1 8 9 7
8 3 1 4 6 7 9 2 5
2 7 6 9 1 3 5 4 8
4 8 7 1 9 5 3 6 2
3 1 9 8 7 2 4 5 6
7 9 3 2 5 6 1 8 4
1 6 8 5 3 4 2 7 9
9 5 4 7 2 8 6 3 1
5 4 2 6 8 9 7 1 3
-------------------
search another Soduku
Yeah!!!find a Soduku
7 2 5 3 4 1 6 9 8
8 3 1 4 5 6 9 2 7
2 7 6 9 1 8 5 4 3
4 8 9 6 2 5 3 7 1
9 1 2 8 7 3 4 5 6
6 5 3 2 9 7 1 8 4
3 6 7 5 8 4 2 1 9
1 9 4 7 6 2 8 3 5
5 4 8 1 3 9 7 6 2
-------------------
search another Soduku
Yeah!!!find a Soduku
9 2 5 3 4 1 6 7 8
8 3 1 4 5 6 9 2 7
2 7 6 8 1 3 5 9 4
4 8 9 7 2 5 3 6 1
3 1 8 9 7 2 4 5 6
7 9 3 2 6 4 1 8 5
1 6 7 5 3 8 2 4 9
6 5 4 1 9 7 8 3 2
5 4 2 6 8 9 7 1 3
-------------------
search another Soduku
```