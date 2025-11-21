---
layout: post
title: "数据结构与算法(1)：并查集"
date: 2025-11-21 20:54:00 +0800
categories: [算法]
math: true
mermaid: false
pin: false
tags: [C++, 并查集, 洛谷, 动态规划]
---

这是数据结构与算法系列的第一篇文章，主要记录学习过程而非知识讲解，以后的文章格式大概就是先学习一种数据结构或算法，结尾再加上一道水题。

## 并查集的引入

[参考文献](https://zhuanlan.zhihu.com/p/93647900)

先看一个题目 [洛谷 P1551](https://www.luogu.com.cn/problem/P1551)。

> **题目描述**
> 
> 规定：$x$ 和 $y$ 是亲戚，$y$ 和 $z$ 是亲戚，那么 $x$ 和 $z$ 也是亲戚。如果 $x$，$y$ 是亲戚，那么 $x$ 的亲戚都是 $y$ 的亲戚，$y$ 的亲戚也都是 $x$ 的亲戚。
> 
> **输入格式**
> 
> 第一行：三个整数 $n,m,p$，（$n,m,p \le 5000$），分别表示有 $n$ 个人，$m$ 个亲戚关系，询问 $p$ 对亲戚关系。
> 
> 以下 $m$ 行：每行两个数 $M_i$，$M_j$，$1 \le M_i,~M_j\le n$，表示 $M_i$ 和 $M_j$ 具有亲戚关系。
> 
> 接下来 $p$ 行：每行两个数 $P_i,P_j$，询问 $P_i$ 和 $P_j$ 是否具有亲戚关系。
> 
> **输出格式**
> 
> $p$ 行，每行一个 `Yes` 或 `No`。表示第 $i$ 个询问的答案为“具有”或“不具有”亲戚关系。

看到这个题，我的第一想法是，开一个桶 `b` ，下标表示该人，值表示其上级亲戚。初始时，令 `b[i]=i` ，读取亲戚关系时进行 `b[mj]=b[mi]` 操作。查询时则将双方依次向上寻根（ 编写 `root` 函数实现），判断根是否相等，`root` 函数实现如下：

```cpp
int root(int c) {
    while(b[c]!=c) c=b[c];
    return c;
}
```

测试样例 AC，提交，果然 WA 声一片，还有一个 TLE 。再回头审视代码，尝试找出反例，发现一组：

```
3 2 1
1 3
2 3
1 2
```

我们模拟 `b` 的情况：

- `1 3`，此时 `b={1,2,1}`
- `2 3`，此时 `b={1,2,2}`
- 查询 `1 2`，显然返回的是 `No`

这说明直接指向上级的方式可能形成孤立的子树。改进方法是让节点直接指向根节点： `b[root(mj)] = root(mi)`，再次提交，AC

## 并查集

事实上，上面的解法已经有了并查集的雏形了，我们来看看标准的 [并查集](https://oi-wiki.org/ds/dsu/)。

```cpp
struct dsu {
  vector<size_t> pa;

  explicit dsu(size_t size) : pa(size) { iota(pa.begin(), pa.end(), 0); }
};
```

可见标准的并查集在初始化时使用了 `iota` 函数：

> Fills the range [`first`, `last`) with sequentially increasing values, starting with `value` and repetitively evaluating `++value`.
> ```cpp
> void iota( ForwardIt first, ForwardIt last, T value );
> ```

而且在查询时可以进行路径压缩：

```cpp
size_t dsu::find(size_t x) { return pa[x] == x ? x : pa[x] = find(pa[x]); }
```

然后是合并操作：
```cpp
void dsu::unite(size_t x, size_t y) { pa[find(x)] = find(y); }
```

至于启发式合并（按秩合并），等到以后需要时再学习吧。

## 水题部分

最后做一道动态规划水题 [洛谷 P1216](https://www.luogu.com.cn/problem/P1216)

思路很简单，直接上核心代码：

```cpp
#include <bits/stdc++.h>
#define rep(i,a,b) for(int i=a; i<=b; i++)

using namespace std;

int a[1002][1002];
int dp[1002][1002];

signed main() {
    cin.tie(nullptr) -> ios::sync_with_stdio(false);
    int r;
    cin >> r;
    rep(i,1,r) rep(j,1,i) cin >> a[i][j], dp[i][j]=max(dp[i-1][j],dp[i-1][j-1])+a[i][j];
    int ans=-1;
    rep(i,1,r) ans=max(ans,dp[r][i]);
    cout << ans;
    return 0;
}
```