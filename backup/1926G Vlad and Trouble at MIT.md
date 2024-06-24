## 原题链接
[Vlad and Trouble at MIT](https://codeforces.com/contest/1926/problem/G)

## 题目大意
学生寝室构成一棵树，有的寝室在开派对（用 $P$ 表示），有的寝室的人要睡觉怕吵（用 $S$ 表示），有的寝室的人无所谓（用 $C$ 表示）。派对的音乐声可以穿墙（通过边传到树的任意节点）。现在要将一些墙体加厚隔断音乐使要睡觉的同学安心睡觉，求最少需要加厚的墙体数量。

## 思路一：dfs
用 $dfs(x, fa, op)$ 表示根为 $x$ 的子树需要加厚的墙体数量，$op$ 表示 $x$ 到其父节点的边是否切断。此时需要对边所连接的节点进行分类讨论。

- 一个 $C$ 一个 $P$：$op = 1$

- 两个节点相同：$op = 0$

- 子节点是 $C$ 但父节点不是 $C$ ：尝试将子节点变为 $S$ 或 $P$ 然后取最小值。

这样便产生了一个问题, 如下图

![](https://cdn.luogu.com.cn/upload/image_hosting/r6yd9hxo.png)

在 $C$ 很多时，复杂度将接近 $O(2^n)$ 这是我们不能接受的。

## 思路二：缩点+贪心
在思路一中，我们注意到增加复杂度的因素是有很多连通的节点C。

如果两个相连通的节点的属性相同，加厚它们之间的边似乎是没有必要的，由此我们想到了可以进行缩点。

缩点的方法也很简单，进行一遍 $dfs$ 。若子节点和父节点不同，给子节点一个新的编号，并对新编号重新连边即可。

```cpp
char a[MAXN], b[MAXN];
int nid[MAXN];
// 缩点
int dfsn = 1; // 1 ~ dfsn
void dfs(int x, int fa)
{
    for (auto y : e[x])
    {
        if (y == fa) continue;
        if (a[y] == a[x])
        {
            nid[y] = nid[x];
            dfs(y, x);
        }
        else
        {
            nid[y] = ++dfsn;
            ee[nid[x]].push_back(nid[y]);
            ee[nid[y]].push_back(nid[x]);
            if (a[x] == 'S' && a[y] == 'P') ans++;
            if (a[x] == 'P' && a[y] == 'S') ans++;
            dfs(y, x);
        }
    }
    b[nid[x]] = a[x];
}
```
在缩点完成之后，我们惊奇地发现好像可以用贪心来解决了。

- 对于 $S$ 和 $P$ 之间的边，直接`ans++`

- 对于和 $C$ 相连的边，分别考虑每个 $C$ 节点所连的边。

	若边 $C-P$ 比边 $C-S$ 要少，就把 $ans$ 加上到边 $C-P$ 的数量，反之亦然。
    
```cpp
dfs(1, 1);
// S-P边在dfs中已经判断过
rep(i, 1, dfsn)
{
    if (b[i] == 'C')
    {
        int s = 0, p = 0;
        for (auto j : ee[i])
        {
            if (b[j] == 'S') s++;
            if (b[j] == 'P') p++;
        }
        if (s > p) ans += p;
        else ans += s;
    }
}
cout << ans << endl;
```
交上去成功WA了，因为有如下反例：

![](https://cdn.luogu.com.cn/upload/image_hosting/tmz3d6ab.png)

## 思路三：树形dp
上图让我们认识到缩点是有问题的，因此还是要回归到 dp 去解决，但是对于C点的处理与思路一不同。

我觉得本场cf的[官方题解](https://codeforces.com/blog/entry/126132)中给出的比喻很不错，这里我给出中文概括：

> 我们想象S节点涌现红色的水，P节点涌现蓝色的水，此时我们把树上下翻转，水沿着边自由向下流，我们的目的是阻断一些边使得两种颜色的水不能混到一起。

我们用 $dp_{i, 1}, dp_{i,2},dp_{i,3}$ 分别表示节点 $i$ 充满蓝色的水、红色的水以及没水时所需隔断的边的最小数量，由叶子节点向根节点逐层进行状态转移，这样最后的结果就是 $min\{dp_{1,1},dp_{1,2},dp_{1,3}\}$。有如下转移方程：(v是u的子节点）

$dp_{u,1} = \sum min\{dp_{v,1}, dp_{v,2}+1, dp_{v,3}\}$

$dp_{u,2} = \sum min\{dp_{v,1}+1, dp_{v,2}, dp_{v,3}\}$

$dp_{u,1} = \sum min\{dp_{v,1}+1, dp_{v,2}+1, dp_{v,3}\}$

由于不同颜色的水不能混合，有颜色的水不能为空，我们将 $dp_u$ 的一些值初始设成无穷大，具体见代码。

```cpp
#include <bits/stdc++.h>
#define MAXN 100010
#define inf 0x3f3f3f3f
#define rep(i, a, b) for(int i = (a); i <= (b); i++)
#define per(i, a, b) for(int i = (a); i >= (b); i--)
using namespace std;
typedef long long ll;
vector <int> e[MAXN];
ll dp[MAXN][5], n;
char a[MAXN];
// p 1, s, 2, c 3;
void dfs(int x)
{
    rep(i, 1, 3) dp[x][i] = 0;
    // 不同颜色的水不能混合，有颜色的水不能为空
    if (a[x] == 'P')
        dp[x][2] = inf, dp[x][3] = inf;
    else if (a[x] == 'S')
        dp[x][1] = inf, dp[x][3] = inf;
        
    for (auto y : e[x])
    {
        // 题目数据保证了小编号是大编号的根，因此建的是有向图
        dfs(y);
        dp[x][1] += min({dp[y][1], dp[y][2] + 1, dp[y][3]});
        dp[x][2] += min({dp[y][1] + 1, dp[y][2], dp[y][3]});
        dp[x][3] += min({dp[y][1] + 1, dp[y][2] + 1, dp[y][3]});
    }
}
int main()
{
    ios::sync_with_stdio(false), cin.tie(nullptr);
    int T; cin >> T;
    for (int ttt = 0; ttt < T; ttt++)
    {
        cin >> n;
        rep(i, 1, n) e[i].clear();
        int tmp;
        rep(i, 2, n)
        {
            cin >> tmp;
            e[tmp].push_back(i);
        }
        rep(i, 1, n) cin >> a[i];
        dfs(1);
        cout << min({dp[1][1], dp[1][2], dp[1][3]}) << endl;;
    }
    return 0;
}
```
[AC记录](https://codeforces.com/contest/1926/submission/247462688)
