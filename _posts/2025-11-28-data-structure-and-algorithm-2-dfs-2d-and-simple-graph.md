---
layout: post
title: "数据结构与算法(2)：二维 DFS 和简单无向图 DFS"
date: 2025-11-28 19:07:00 +0800
categories: [算法]
math: true
mermaid: false
pin: false
tags: [C++, 搜索, 洛谷, DFS, 递归, 无向图]
---

本文介绍搜索算法的基础（之一）——DFS，DFS 是基于递归的，也就是“不撞南墙不回头”。本文将以例题的形式讲解二维 DFS 和简单无向图 DFS。

## 二维 DFS

二维 DFS 其实就是在一个二维网格（矩阵）上进行的 DFS。通常我们从网格中的一个点开始，向四个方向或八个方向扩展，访问相邻的单元格，直到所有连通区域都被访问，常用于解决迷宫问题、连通区域标记、填色问题等。

### [例 1：填涂颜色（洛谷 P1162）](https://www.luogu.com.cn/problem/P1162)

给定一个 $n\times n$ 的矩阵，包含 $0$ 和 $1$，要求将所有被 $1$ 包围的 $0$ 替换为 $2$。关键思路是从边界开始 DFS，标记所有与边界连通的 $0$，剩下的未被标记的 $0$ 就是被包围的区域。

{% raw %}
```cpp
// P1162 填涂颜色
// CrispyXYZ
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=a; i<=b; i++)
#define each(e,a) for(auto &&e: a)
#define all(v) v.begin(),v.end()
#define comp() [](auto &a, auto &b)
#define endl '\n'

using PII = std::pair<int,int>;
using VI = std::vector<int>;
using LL = long long;
using namespace std;

int n;
int a[35][35];

bool flag[35][35];
vector<PII> d{{1,0},{-1,0},{0,1},{0,-1}};

void dfs(int i, int j) {
    if(a[i][j]==1||flag[i][j]||i==0||i==n+1||j==n+1||j==0) return;
    flag[i][j]=true;
    each(p,d) dfs(i+p.first, j+p.second);
}

signed main() {
    cin.tie(nullptr) -> ios::sync_with_stdio(false);
    cin >> n;
    rep(i,1,n) rep(j,1,n) cin >> a[i][j];
    rep(i,1,n) rep(j,1,n) if(i==1||j==1||i==n||j==n) dfs(i,j);
    rep(i,1,n) {
        rep(j,1,n) {
            cout << (flag[i][j]||a[i][j]==1? a[i][j] : 2) << " ";
        }
        cout << endl;
    }

    return 0;
}
```
{% endraw %}

**代码简析：**

- 用数组 `d` 定义了四个移动方向：下、上、右、左。用于简化代码（这个是二维 DFS 通用的）
- `dfs` 函数的边界条件：遇到墙（1）、已访问、越界时返回
- 核心思路：从所有边界点开始 DFS，标记所有能到达的 0，最后未被标记的 0 就是被包围的区域
- 输出时：已标记或原值为 1 的输出原值，否则输出 2

### [例 2：求细胞数量（洛谷 P1451）](https://www.luogu.com.cn/problem/P1451)

计算矩阵中连通区域的个数。每个细胞由相邻的非零数字组成。使用 DFS 遍历整个矩阵，遇到未访问的非零单元格就进行 DFS 标记整个连通区域。

{% raw %}
```cpp
// P1451 求细胞数量
// CrispyXYZ
#include <bits/stdc++.h>
#include <cmath>
#define rep(i,a,b) for(int i=a; i<=b; i++)
#define each(e,a) for(auto &&e: a)
#define all(v) v.begin(),v.end()
#define comp() [](auto &a, auto &b)
#define endl '\n'

using PII = std::pair<int,int>;
using VI = std::vector<int>;
using LL = long long;
using namespace std;

int m,n;
char mat[105][105];
bool f[105][105];
int cnt=0;
vector<PII> d{{1,0},{0,1},{-1,0},{0,-1}};

void dfs(int i, int j) {
    if(f[i][j]||mat[i][j]=='0'||i==0||j==0||i==m+1||j==n+1) return;
    f[i][j] = true;
    each(e,d) dfs(i+e.first,j+e.second);
}

signed main() {
    cin.tie(nullptr) -> ios::sync_with_stdio(false),
    cin >> m >> n;
    rep(i,1,m) rep(j,1,n) cin >> mat[i][j];
    rep(i,1,m) rep(j,1,n) if(!f[i][j]&&mat[i][j]!='0') cnt++,dfs(i,j);
    cout << cnt;
    return 0;
}
```
{% endraw %}

**代码解析：**

- 使用字符矩阵存储输入，`'0'` 表示背景，非 `'0'` 表示细胞
- `f` 数组记录每个位置是否被访问过
- 遍历每个单元格，如果发现未访问的细胞就计数并 DFS 标记整个连通区域
- DFS 过程中跳过已访问、背景和越界的位置

## 简单无向图 DFS

简单无向图 DFS 常见于简单的路径查找等问题，通常使用邻接矩阵表示图，递归遍历节点。

### [例 3：高手去散步（洛谷 P1294）](https://www.luogu.com.cn/problem/P1294)

在无向带权图中找到最长的简单路径（不重复经过节点）。从每个节点出发进行 DFS，记录路径长度，使用回溯法尝试所有可能的路径。

```cpp
// P1294 高手去散步
// CrispyXYZ
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=a; i<=b; i++)
#define each(e,a) for(auto &&e: a)
#define all(v) v.begin(),v.end()
#define comp() [](auto &a, auto &b)
#define endl '\n'

using PII = std::pair<int,int>;
using VI = std::vector<int>;
using LL = long long;
using namespace std;

int n,m;
int ans;
int graph[25][25];
bool v[25];

void dfs(int node, int len) {
    ans = max(ans,len);
    v[node]=true;
    rep(i,1,n) if(!v[i]&&graph[node][i]) dfs(i,len+graph[node][i]);
    v[node]=false;
}

signed main() {
    cin.tie(nullptr) -> ios::sync_with_stdio(false);
    cin >> n >> m;
    int a,b,l;
    rep(i,1,m) {
        cin >> a >> b >> l;
        graph[a][b]=graph[b][a]=l;
    }
    int ma=0;
    rep(i,1,n) {
        ans=0;
        dfs(i,0);
        ma=max(ma,ans);
        memset(v,0,sizeof(v));
    }
    cout << ma;

    return 0;
}
```

**代码解析：**

- 使用邻接矩阵 `graph` 存储无向图
- `v` 数组记录节点访问状态，避免重复访问
- `dfs` 函数参数：当前节点和当前路径长度
- 关键技巧：回溯时重置访问标记 `v[node]=false`，从而允许其他路径重新访问该节点
- 从每个节点出发进行 DFS，找到全局最长路径

## 总结

DFS 核心思想总结：

- 递归实现，注意终止条件
- 需要标记已访问状态避免重复
- 二维 DFS 注意边界检查
- 图 DFS 注意回溯时重置状态