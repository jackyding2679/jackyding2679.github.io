---
layout: post
title: '算法经典-八皇后问题'
date: 2020-02-29
author: Jacky
categories: 经典算法
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 回溯算法 八皇后问题
---

>

### 提出问题
有个8*8的棋盘，需要在每一行放置一个棋子，
每个棋子横，竖，斜着方向都没有其他棋子，要求输出结果

例如，如下有一个4*4的棋盘，#代表一个棋子，满足每行放置了一个棋子，每个棋子的横，竖，斜
着的方向都没有其他棋子的要求
```css
* # * *
* * * #
# * * *
* * # *
```