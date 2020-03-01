---
layout: post
title: '算法经典-旅行商问题'
date: 2020-03-01
author: Jacky
categories: 经典算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 回溯算法 旅行商问题
---

使用回溯算法解决旅行商问题。

### 提出问题
一个售货员必须访问n个城市，这n个城市是一个完全图，
售货员需要恰好访问所有城市的一次，并且回到最终的城市。
城市于城市之间有一个距离，售货员希望旅行距离最短。


### 算法实现
```clike
//旅行商（Traveling salesman problem, TSP)
//一个售货员必须访问n个城市，这n个城市是一个完全图，
//售货员需要恰好访问所有城市的一次，并且回到最终的城市。
//城市于城市之间有一个旅行费用，售货员希望旅行费用之和最少。
//Author: jackyding
//Date: 2020/03/01
//Description: solve traveling-salesman-problem

#include <stdio.h>
#include <string.h>

#define GRAPH_SIZE 5//图(graph)顶点(vetex)个数

//为了方便，使用二维数组存储整个图，为邻接矩阵（adjacency matrix)
//顶点的id为0，1，2，3，二维数组的值表示顶尖之间的距离，顶点跟自己的距离
//为0,例如
//DISTANCE(V0,V2) = graph[0][2]
//DISTANCE(V2,V0) = graph[2][0]
const int graph[GRAPH_SIZE][GRAPH_SIZE] = {
    {0, 6, 4, 1, 2},
    {6, 0, 5, 4, 3},
    {4, 5, 0, 5, 1},
    {1, 4, 5, 0, 1},
    {2, 3, 1, 1, 0}
};

int min_distance = 0;//存储最小距离
int shortest_path[GRAPH_SIZE];//存储最终路径
#if 0
//也可以使用邻接表（adjacency list)存储图
//定义顶点
struct Vetex {
    int id;//id唯一标识一个顶点
    int distance;//顶点之间的距离
};

//定义单链表(linked list)
struct Node;
typedef struct Node *PtrNode;
typedef PtrNode List;
typedef PtrNode Position;

struct Node {
    struct Vetex v;
    Position next;
};

List graph[GRAPH_SIZE];//头结点不存储数据
#endif

void PrintPath(int *p, int s) {
    printf("path is: ");
    for (int i = 0; i < s; i++) {
        printf("%d", p[i]);
        if (i != (s - 1)) {
            printf("->");
        }
    }
    printf("\n-------------------\n");
}

//v,顶点ID
//p,存储顶点
//s,顶点个数
//d,路径长度
void TSP(int v, int *p, int s, int d) {
    int i, j;

    if (s == GRAPH_SIZE) {
        if (min_distance == 0 || d < min_distance) {
            printf("set min_distance to %d\n", d);
            min_distance = d;
            PrintPath(p, s);
            memcpy(shortest_path, p, sizeof(shortest_path));
        } else {
            printf("d(%d) >= min_distance(%d)\n", d, min_distance);
            PrintPath(p, s);
        }
        return;
    }
    for (i = 0; i < GRAPH_SIZE; i++) {
        bool next;
        if (i == v) {
            //顶点自己,不能选
            continue;
        }

        for (j = 0; j < s; j++) {
            //已经选择的顶点，不能再被选择
            if (p[j] == i) {
                next = true;
                break;
            }
        }
        if (next) {
            next = false;
            continue;
        }

        if (s < (GRAPH_SIZE - 1) && (i == 0)) {
            //在其他节点都选择完之前，不能选择V0
            continue;
        }

        if (min_distance != 0 && (d + graph[v][i] >= min_distance)) {
            //剪枝
            continue;
        }

        p[s] = i;
        s++;
        TSP(i, p, s, d + graph[v][i]);
        --s;
    }
    return;
}

int main() {
    int p[GRAPH_SIZE];
    TSP(0, p, 0, 0);
    printf("min distance is %d\n", min_distance);
    PrintPath(shortest_path, GRAPH_SIZE);
    return 0;
}
```
### 输出结果
```clike
[root@node1 backtracking]# ./Traveling-Salesman-Problem
set min_distance to 19
path is: 1->2->3->4->0
-------------------
set min_distance to 14
path is: 1->2->4->3->0
-------------------
d(18) >= min_distance(14)
path is: 1->3->2->4->0
-------------------
d(16) >= min_distance(14)
path is: 1->3->4->2->0
-------------------
d(16) >= min_distance(14)
path is: 1->4->2->3->0
-------------------
d(19) >= min_distance(14)
path is: 1->4->3->2->0
-------------------
d(16) >= min_distance(14)
path is: 2->1->3->4->0
-------------------
d(14) >= min_distance(14)
path is: 2->1->4->3->0
-------------------
d(18) >= min_distance(14)
path is: 2->3->1->4->0
-------------------
d(19) >= min_distance(14)
path is: 2->3->4->1->0
-------------------
set min_distance to 13
path is: 2->4->1->3->0
-------------------
d(16) >= min_distance(13)
path is: 2->4->3->1->0
-------------------
d(13) >= min_distance(13)
path is: 3->1->2->4->0
-------------------
d(13) >= min_distance(13)
path is: 3->1->4->2->0
-------------------
d(16) >= min_distance(13)
path is: 3->2->1->4->0
-------------------
d(16) >= min_distance(13)
path is: 3->2->4->1->0
-------------------
d(14) >= min_distance(13)
path is: 3->4->1->2->0
-------------------
d(14) >= min_distance(13)
path is: 3->4->2->1->0
-------------------
d(16) >= min_distance(13)
path is: 4->1->2->3->0
-------------------
d(18) >= min_distance(13)
path is: 4->1->3->2->0
-------------------
d(13) >= min_distance(13)
path is: 4->2->1->3->0
-------------------
d(18) >= min_distance(13)
path is: 4->2->3->1->0
-------------------
d(16) >= min_distance(13)
path is: 4->3->1->2->0
-------------------
d(19) >= min_distance(13)
path is: 4->3->2->1->0
-------------------
min distance is 13
path is: 2->4->1->3->0
-------------------
```