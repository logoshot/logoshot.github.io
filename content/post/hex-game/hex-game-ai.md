---
title: "Hex Game Ai"
date: 2022-11-25T19:54:28+08:00
draft: false
---

> 好久好久没有写blog了，最近很认真的在研究hex的ai，研究mcts，想要记录一下自己曾经的努力，不然一年到头总觉得一事无成。

# 资料

## 教程

- [如何学习蒙特卡罗树搜索（MCTS）](# 如何学习蒙特卡罗树搜索（MCTS）)

- [深入浅出解读并思考AlphaGo](https://zhuanlan.zhihu.com/p/352661568)

## 代码示例

- [monte-carlo-tree-search](https://github.com/int8/monte-carlo-tree-search.git/https://github.com/int8/monte-carlo-tree-search.git/)

- [Awesome Monte Carlo Tree Search Papers](https://github.com/benedekrozemberczki/awesome-monte-carlo-tree-search-papers)

- [alpha-zero-general](https://github.com/suragnair/alpha-zero-general.git) A clean implementation based on alphazero for any game

- [alpha-zero-gomoku](https://github.com/hijkzzz/alpha-zero-gomoku) A multi-thread implementation of alphazero(c++)

- [AlphaZero_Gomoku](https://github.com/junxiaosong/AlphaZero_Gomoku) (python)

## 论文

- A Survey of Monte Carlo Tree Search Methods

## 游戏网站

- [hex game(with ai)](https://lutanho.net/play/hex.html)

# 编程实现

在魔改[alpha-zero-general](https://github.com/suragnair/alpha-zero-general.git)的代码的过程中遇到的问题，首先第一个问题就是由于hex是六边形的棋格，所以棋子的相邻关系在棋盘旋转九十度后会发生变化，所以原代码中的使用到旋转90度的地方需要修改，也就是getsym里生成对称棋盘的地方。又由于规则已经限定了红色和蓝色的边各是哪两条，因此获取棋盘的canonicalform的时候，就不能简单的用棋盘乘上-1来实现。应当使用对角线的镜像翻转再乘上-1得到。
