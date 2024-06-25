# 原题链接
[Array Optimization by Deque](https://codeforces.com/contest/1579/problem/E2)

# 题目大意
题目要求处理一个整数数组，并将其元素逐一添加到一个空的双端队列（deque）中。在添加每个元素之前，可以选择将其添加到队列的开头或结尾。目标是找到在添加完所有数组元素后，队列中逆序对的最小数量。

# 思路简述
1. **数据离散化**：首先对数组元素进行排序，并通过排序索引实现数据的离散化处理。
2. **树状数组优化**：使用树状数组来优化逆序对的计数过程，实现快速的单点更新和区间求和。
3. **逆序对计算**：对于数组中的每个元素，计算将其添加到队列开头或结尾时新增的逆序对数量，并选择使新增逆序对数量最小的方案。把一个数 $x$ 添加进双端队列相当于给 $c[x]$ 加一，求逆序对相当于求c[1]~c[x-1]与c[x+1]~c[N]之和，其中N为离散化之后最大的数。

# 代码实现
```cpp
#include <bits/stdc++.h>
#define MAXN 200010
#define inf 0x3f3f3f3f
#define rep(i, a, b) for(int i = (a); i <= (b); i++)
#define per(i, a, b) for(int i = (a); i >= (b); i--)
using namespace std;
typedef long long ll;
int N;
struct node{
    int id;
    int val;
    bool operator<(const node &b) const {
        return val < b.val;
    }
}a[MAXN];
bool cmp(node a, node b) {
    return a.id < b.id;
}
int lowbit(int x) {
    return x & -x;
}
void update(int i, int val, vector<int> &v) {
    while (i <= N) {
        v[i] += val;
        i += lowbit(i);
    }
}
int sum(int i, vector<int> &v) {
    int res = 0;
    while (i > 0) {
        res += v[i];
        i -= lowbit(i);
    }
    return res;
}
int range_sum(int l, int r, vector<int> &v) {
    return sum(r, v) - sum(l - 1, v);
}
void print(vector<int> &v) {
    for (auto i : v) {
        cout << i << " ";
    }
    cout << endl;
}
int main()
{
    ios::sync_with_stdio(false), cin.tie(nullptr);
    int T; cin >> T;
    for (int ttt = 0; ttt < T; ttt++)
    {
        int n; cin >> n;
        rep(i, 1, n) {
            int x; cin >> x;
            a[i].id = i;
            a[i].val = x;
        }
        sort(a + 1, a + 1 + n);
        int cnt = 0;
        a[0].val = -inf;
        vector<int> b(n + 5, 0);
        for (int i = 1; i <= n; i++) {
            if (a[i - 1] < a[i]) {
                b[i] = ++cnt;
            } else {
                b[i] = cnt;
            }
        }
        rep(i, 1, n) a[i].val = b[i];
        sort(a + 1, a + 1 + n, cmp);
        N = cnt;
        vector<int> c(n + 5, 0);
        ll ans = 0;
        for(int k = 1; k <= n; k++) {
            int i = a[k].val;
            update(i, 1, c);
            // cout << i << " " << range_sum(1, i - 1, c) << " " << range_sum(i + 1, N, c) << endl;
            ans += min(range_sum(1, i - 1, c), range_sum(i + 1, N, c));
        }
        cout << ans << endl;
    }
    return 0;
}
```