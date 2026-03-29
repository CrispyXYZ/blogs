---
layout: post
title: "数据结构与算法(3)：从迷宫 BFS 到双端队列 0-1 BFS"
date: 2026-03-25 18:53:00 +0800
categories: [算法]
math: true
mermaid: false
pin: false
tags: [C++, 搜索, BFS, 队列, 二维]
---

~~时隔一个多月终于迎来了久违的更新~~

众所周知，DFS 和 BFS 都是常用的基础搜索算法，其中 DFS 常常使用栈结构做储存，而 BFS 常常使用队列结构做储存。两种算法都在寻路搜索中有一定的应用。

本文将从一道算法题出发，从经典的迷宫寻路 BFS 进阶到 0-1 BFS 算法。

## 回顾经典的迷宫最短路径 BFS

题目来源：[U425042 迷宫最短路径](https://www.luogu.com.cn/problem/U425042)

### 题目背景

- 小明是一个寻宝爱好者，最近他听说了一个关于迷宫中藏有宝藏的传说。于是，他决定挑战这个迷宫，找到隐藏在其中的宝藏。但是，迷宫中布满了墙壁和通道，而且迷宫中还隐藏了许多陷阱，想要顺利找到宝藏并不容易。

### 题目描述

- 给定一个迷宫地图，地图由一个二维数组表示，数组中的每个元素代表一个迷宫单元格，其中 $0$ 表示可通行的空地， $1$ 表示墙壁。小明的任务是找到从迷宫的起点到终点的最短路径，并输出路径的长度。迷宫的起点用 $S$ 表示，终点用 $E$ 表示。小明可以在迷宫中向上、向下、向左、向右移动，但不能穿过墙壁。

### 输入格式

- 第一行包含两个整数 $n$ 和 $m$ ，表示迷宫的行数和列数。
- 接下来的 $n$ 行，每行包含 $m$ 个整数，描述迷宫的地图。其中 $0$ 表示空地， $1$ 表示墙壁， $S$ 表示起点， $E$ 表示终点。

### 输出格式

- 输出一个整数，表示从起点到终点的最短路径的长度。如果不存在从起点到终点的路径，则输出 $-1$ 。

### 输入输出样例 #1

#### 输入 #1

```
5 5
S 0 0 0 0
1 1 1 1 0
0 0 1 0 0
0 0 0 0 1
1 1 1 0 E
```

#### 输出 #1

```
10
```

### 输入输出样例 #2

#### 输入 #2

```
4 4
1 0 0 0
1 1 1 1
0 0 1 S
0 0 0 E
```

#### 输出 #2

```
1
```

### 说明/提示

- 对于 $100\%$ 的数据， $1 \leq n \leq 100, 1 \leq m \leq 100$ 。

### 题解

应用经典的 BFS 算法，采用 `vector<vector<char>> visited` 二维数组标记访问过的位置，`vector<vector<int>> dist` 二维数组标记距离（初始值 $-1$），`queue<pair<int,int>> q` 队列存放当前待扩展的坐标。初始时将起点坐标入队，循环中遍历队首，使其出队，再对其四个方向入队，标记 `visited` 并更新 `dist`。

由于 BFS 的算法特性，先访问到的位置一定是路径最短的，故当访问到终点时直接输出并终止程序即可。

代码：

{% raw %}
```cpp
// U425042 迷宫最短路径
// CrispyXYZ
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=a; i<=b; i++)
#define each(e,a) for(auto &&e: a)
#define all(v) v.begin(),v.end()
#define comp() [](auto &a, auto &b)
#define endl '\n'

using LL = long long;
using PII = std::pair<int,int>;
using VI = std::vector<int>;
using namespace std;

int n,m,sx,sy,ex,ey;

bool isValid(int x, int y) {
    return (0<=x&&x<n)&&(0<=y&&y<m);
}

vector<vector<int>> dir = {{0,1}, {0,-1}, {1,0}, {-1,0}};

signed main() {
    cin >> n >> m;
    vector<vector<char>> mat(n, vector<char>(m));
    vector<vector<char>> visited(n, vector<char>(m,0));
    vector<vector<int>> dist(n, vector<int>(m,-1));
    queue<PII> q;
    rep(i,0,n-1) rep(j,0,m-1) {
        char tmp;
        cin >> tmp;
        mat[i][j]=tmp;
        if (tmp=='S') sx=i, sy=j;
        if (tmp=='E') ex=i, ey=j;
    }
    q.push({sx,sy});
    dist[sx][sy]=0;
    visited[sx][sy]=1;
    while(!q.empty()) {
        int qx=q.front().first;
        int qy=q.front().second;
        q.pop();
        rep(i,0,3) {
            int nx = qx+dir[i][0];
            int ny = qy+dir[i][1];
            if (isValid(nx,ny) && mat[nx][ny]!='1' && visited[nx][ny]!=1) {
                if (nx==ex && ny==ey) {
                    cout << dist[qx][qy]+1;
                    return 0;
                }
                visited[nx][ny]=1; // 注：visited 用于控制该坐标是否入队，故判断后应当置 visited 为 1
                q.push({nx,ny});
                dist[nx][ny] = dist[qx][qy]+1;
            }
        }
    }
    cout << -1;
    return 0;
}
```
{% endraw %}

## 进阶：0-1 BFS

若是迷宫无路可走（没有到达终点的**地面**通路），你会怎么办？那我们是不是可以尝试翻墙 ~~（字面意思）~~ ？翻墙需要耗费体力，所以不到万不得已，你不会轻易尝试翻墙。在这个新的视角下，地面上的移动不会耗费体力，而从地面翻上墙（以及从墙上跳到地面）会耗费 $1$ 点体力。（同时在墙上的通路中通行也不耗费体力。）我们很自然的就会想要求出从起点到终点耗费的最小体力（此时路径长度已经不重要了）。这样就构造出了另一种 BFS 问题：0-1 BFS。

于是这是，我们要 BFS 搜索的对象就从路径长度，转变为体力耗费点数。在最短路径问题中，我门总是在队列中先推入长度为 $1$ 的路径（并检查），再推入长度为 $2$ 的路径，以此类推；同理，我们有一个很自然的想法，即，在 0-1 BFS 中，总是先处理体力耗费为 $0$ 的，之后是 $1$，以此类推。

如何高效的实现后者呢？有一个很直观但不够高效的方法，先记录当前所有消耗为 $0$ 的坐标，若没有找到终点，则向四周扩散寻找所有消耗为 $1$ 的坐标，以此类推。这种方式的开销主要在于标记了所有 $0$ 而不是边界处的 $0$ ，导致在搜索 $1$ 时有冗余的搜索，造成重复计算，浪费了算力。

于是，我们提出想法：如果在搜索 $0$ 时顺手就把边界 $0$ 记录下来呢？经过简单分析，不难验证这样是可以实现高效搜索的。不过，刻意记录 $0$ 仍然存在一定的冗余，不妨在搜索时，将边界 $0$ 的相邻 $1$ 存入另一个队列。

我们来看一道 0-1 BFS 的经典问题。

### Description

小明最近喜欢玩一个游戏。给定一个 $n \times m$ 的棋盘，上面有两种格子 `#` 和 `@`。游戏的规则很简单：给定一个起始位置和一个目标位置，小明每一步能向上，下，左，右四个方向移动一格。如果移动到同一类型的格子，则费用是 $0$，否则费用是 $1$。请编程计算从起始位置移动到目标位置的最小花费。

### Input

输入文件有多组数据。

输入第一行包含两个整数 $n$，$m$，分别表示棋盘的行数和列数。

输入接下来的 $n$ 行，每一行有 $m$ 个格子（使用 `#` 或者 `@` 表示）。

输入接下来一行有四个整数 $x_1$, $y_1$, $x_2$, $y_2$，分别为起始位置和目标位置。

当输入 $n$，$m$ 均为 $0$ 时，表示输入结束。

### Output

对于每组数据，输出从起始位置到目标位置的最小花费。每一组数据独占一行。

### Sample

Input:

```
2 2
@#
#@
0 0 1 1
2 2
@@
@#
0 1 1 0
0 0
```

Output:

```
2
0
```
### Hint

对于 $100\%$ 的数据满足：$1 \le n, m \le 500$。

### 题解

{% raw %}
```cpp
// B - ????[2009]?????
// CrispyXYZ
#include <bits/stdc++.h>
using namespace std;

bool continues = true;
int n,m;

bool isValid(int x, int y) {
    return (0<=x&&x<n)&&(0<=y&&y<m);
}

vector<vector<int>> dir = {{0,1}, {0,-1}, {1,0}, {-1,0}};


void solve() {
    cin >> n >> m;
    if (n==0 && m==0) {
        continues=false;
        return;
    }
    vector<vector<char>> mat(n,vector<char>(m));
    vector<vector<char>> visited(n, vector<char>(m,0));
    for(int i=0; i<n; i++) for(int j=0; j<m; j++) cin >> mat[i][j];
    int x1,y1,x2,y2;
    cin >> x1 >> y1 >> x2 >> y2;
    queue<pair<int,int>> q1,q2;
    q1.push({x1,y1});
    visited[x1][y1]=0;
    char ch = mat[x1][y1];
    int cnt=-1;
    queue<pair<int,int>> &q = q1;
    queue<pair<int,int>> &oq = q2;
    lbl:
    ++cnt;
    while(!q.empty()) {
        int cx = q.front().first;
        int cy = q.front().second;
        q.pop();
        for(int i = 0; i<=3; i++) {
            int nx = cx+dir[i][0];
            int ny = cy+dir[i][1];
            if(isValid(nx, ny) && !visited[nx][ny]) {
                visited[nx][ny]=1;
                if(mat[nx][ny] == ch) {
                    if(nx==x2 && ny==y2) {
                        cout << cnt << '\n';
                        return;
                    }
                    q.push({nx,ny});
                } else {
                    if(nx==x2 && ny==y2) {
                        cout << (cnt+1) << '\n';
                        return;
                    }
                    oq.push({nx,ny});
                }
            }
        }
    }
    swap(q,oq);
    ch = (ch=='#'?'@':'#');
    goto lbl;
    
}

signed main() {
    while(continues) solve();
    return 0;
}
```
{% endraw %}

### 注意

这里采用两个队列的实现方式并不是标准的 0-1 BFS，标准的 0-1 BFS 采用双端队列实现，可以在一个循环内完成整个二维网络的搜索，代码更简洁，不过时间复杂度不变。
