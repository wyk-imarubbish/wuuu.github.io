## 原题链接：
[Sasha and a Walk in the City](https://codeforces.com/problemset/problem/1929/D)

## 题意简述：
给定一棵无根树，要求在树上选择一些节点（可以不选）将其变为危险的点，使得树上任意一条简单路径上至多只有两个危险的点。问共有多少种选法。

## 思路：子树dp
比较容易想到树上危险点的选法与子树上的选法是有关系的，解决本题的关键在于分类讨论。

### 状态设计
在给出状态的表示之前，我们先思考一个问题。当子树处于什么情况下除了这颗子树外其他的节点中不能再选危险点了？

注意到子树上任意一点到子树外一点的简单路径上一定会经过这颗子树的根节点。如果**存在**子树上的一个点，其到子树的根的路径上有两个危险的点，则这颗子树外一定不能再选取危险的点了。此时我们不妨称这颗子树为**满的**。

与之相对应，若子树上任意一点到子树根节点的路径上危险点的个数都等于 1 则称这颗子树是**不满**的；若子树上一个危险点都没有，则称这颗子树是**空的**。

由此我们便能得到这道题的状态表示 $dp_{u,i}$ ，其中 $u$ 为子树的根节点，表示这颗子树， $i=0$ 时表示子树是**满的**，$i = 1$ 表示子树是**不满**的。
### 边界情况
对于叶子节点，将其视为一棵子树，该节点要么危险要么不危险。

因此，$dp_{v,0}=0,dp_{v,1}=1$

### 转移方程
对于子树 $u$ ，我们先考虑 $dp_{u,0}$ ,即 $u$ 是**满的**。

1. 若 $u$ 选为危险的点，则 $u$ **有且仅有**一棵子树是**不满**的。
2. 若 $u$ 不选，则 $u$ **有且仅有**一棵子树是**满的**。

于是有转移方程：$dp_{u,0} = \sum (dp_{v,0} + dp_{v,1})$

接着，我们考虑 $dp_{u,1}$ ，即 $u$ 是**不满**的。

1. 若 $u$ 选为危险的点, 则以 $u$ 为根的子树中不能再选其他的点，共 1 种选法。
2. 若 $u$ 不选，则 $u$ 的子树可以是**不满**的，也可以是**空的**。根据**乘法原理**可知，共 $\prod (dp_{v,1} + 1)$ 种。但是不能全是空的，因此还要减去 1 。

所以， $dp_{u,1} = 1+\prod (dp_{v,1}+1)-1=\prod (dp_{v,1}+1)$。

### 最终答案
我们选取1为整棵树的根，则最终答案为 $dp_{1,0}+dp_{1,1}+1$ ，其中 1 表示整棵树都是空的。

```cpp
#include <bits/stdc++.h>
#define MAXN 300010
#define MOD 998244353
#define rep(i, a, b) for(int i = (a); i <= (b); i++)
#define per(i, a, b) for(int i = (a); i >= (b); i--)
using namespace std;
typedef long long ll;
int head[MAXN], cnt, n;
ll dp[MAXN][2];
struct Edge
{
    int to, next;
}e[MAXN << 2];
void add_edge(int from, int to)
{
    e[++cnt] = {to, head[from]};
    head[from] = cnt;
}
void dfs(int x, int fa)
{
    dp[x][1] = 1;
    dp[x][0] = 0;
    for (int i = head[x]; i; i = e[i].next)
    {
        int y = e[i].to;
        if (y == fa) continue;
        dfs(y, x);
        dp[x][1] = dp[x][1] * (dp[y][1] + 1) % MOD;
        dp[x][0] = (dp[x][0] + dp[y][0] + dp[y][1]) % MOD;
    }
}
int main()
{
    ios::sync_with_stdio(false), cin.tie(nullptr);
    int T; cin >> T;
    for (int ttt = 0; ttt < T; ttt++)
    {
        memset(head, 0, sizeof(head));
        cnt = 0;
        cin >> n;
        int u, v;
        rep(i, 1, n - 1)
        {
            cin >> u >> v;
            add_edge(u, v);
            add_edge(v, u);
        }
        dfs(1, 1);
        cout << (dp[1][0] + dp[1][1] + 1) % MOD << endl;
    }
    return 0;
}
```