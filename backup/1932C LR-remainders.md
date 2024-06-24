## 原题链接
[LR-remainders](https://codeforces.com/contest/1932/problem/C)

## 题目大意
给你一个长度为 $n$ 的数组 $a$、一个正整数 $m$ 和一串长度为 $n$ 的命令。每条命令要么是字符 $L$，要么是字符 $R$。

按照字符串 $s$ 中的顺序处理所有 $n$ 条命令。处理一条命令的步骤如下：

首先，输出数组 $a$ 中所有元素除以 $m$ 后的乘积余数。然后，如果命令是 $L$，则从数组 $a$ 中删除最左边的元素；如果命令是 $R$，则从数组 $a$ 中删除最右边的元素。注意，每次移动后，数组 $a$ 的长度都会减少 $1$，处理完所有命令后，数组 $a$ 将为空。

请编写一个程序，按照字符串 $s$ 中的顺序(从左到右)处理所有命令。

## 思路一：
维护数组中所有元素的积对 $m$ 的余数，记作 $res$ ，并且用 $l$ 和 $r$ 来维护数组的左右端点，每次操作求解同余方程。

例如：$L$ 操作就相当于求解方程 $a[l] \times x \equiv \pmod k$，然后 `l++` 。

那么问题来了，**如何求解线性同余方程呢？**

参考 [oi-wiki](https://oi-wiki.org/math/number-theory/linear-equation/) 中的介绍，可以用扩展欧几里得算法来进行求解。于是我写出了如下代码：
```cpp
int ex_gcd(int a, int b, int &x, int &y)
{
    if (b == 0)
    {
        x = 1;
        y = 0;
        return a;
    }
    int d = ex_gcd(b, a % b, x, y);
    int tmp = y;
    y = x - (a / b) * y;
    x = tmp;
    return d;
}
int equ(int a, int b, int n)
{
    // ax+ny = b;
    int x, y;
    int d = ex_gcd(a, n, x, y);
    int k = b / d;
    x *= k;
    y *= k;
    return (x + n) % n;
}
```
经过测试，equ 函数确实可以求出方程 $ax \equiv b \pmod n$ 的一个小于 $n$ 的非负整数解。于是，有如下代码：
```cpp
int main()
{
    ios::sync_with_stdio(false), cin.tie(nullptr);
    int n, m;
    int T; cin >> T;
    for (int ttt = 0; ttt < T; ttt++)
    {
        cin >> n >> m;
        int now = 1;
        rep(i, 1, n)
        {
            cin >> a[i];
            now = now * a[i] % m;
        }
        char op;
        int l = 1, r = n;
        rep(i, 1, n)
        {
            cin >> op;
            if (op == 'L')
            {
                // a[l] * x = now (mod m)
                now = equ(a[l], now, m);
                cout << now << ' ';
                l++;
            }
            else
            {
                now = equ(a[r], now, m);
                cout << now << ' ';
                r--;
            }
        }
        cout << endl;
    }
    return 0;
}
```
运行一下，惊奇地发现一个样例都过不去。原因在于：**线性同余方程是有多解的**。我天真地以为根据题目要求可以限制出唯一满足条件的整数解，但事实并非如此。$3x \equiv 0 \pmod 6$ 的解可以是 $0$ 和 $2$ ，此时就不能确定了。

## 思路二（正解）
思路一求解同余方程，本质是想要在取模意义下做除法，这正是导致产生多解的原因。注意到除法的逆运算——乘法，在取模意义下的解是唯一的，于是便想到了倒着做，即从数组只剩一个元素的情况倒推回去，每次都乘以左边或右边的元素，这样就不会产生多解了。另外，可以用栈存储答案，这样输出时的顺序就变回去了。

```cpp
#include <bits/stdc++.h>
#define MAXN 200010
#define rep(i, a, b) for(int i = (a); i <= (b); i++)
#define per(i, a, b) for(int i = (a); i >= (b); i--)
using namespace std;
typedef long long ll;
int ex_gcd(int a, int b, int &x, int &y)
{
    if (b == 0)
    {
        x = 1;
        y = 0;
        return a;
    }
    int d = ex_gcd(b, a % b, x, y);
    int tmp = y;
    y = x - (a / b) * y;
    x = tmp;
    return d;
}
int equ(int a, int b, int n)
{
    // ax+ny = b;
    int x, y;
    int d = ex_gcd(a, n, x, y);
    int k = b / d;
    x *= k;
    y *= k;
    return (x + n) % n;
}
int a[MAXN];
int main()
{
    ios::sync_with_stdio(false), cin.tie(nullptr);
    int n, m;
    int T; cin >> T;
    for (int ttt = 0; ttt < T; ttt++)
    {
        cin >> n >> m;
        stack <int> ans;
        rep(i, 1, n) cin >> a[i];
        string op;
        int l = 1, r = n;
        cin >> op;
        for (auto i : op)
        {
            if (i == 'L') l++;
            else r--;
        }
        int now = 1;
        per(i, n - 1, 0)
        {
            if (op[i] == 'L')
            {
                l--;
                now = now * a[l] % m;
                ans.push(now);
            }
            else
            {
                r++;
                now = now * a[r] % m;
                ans.push(now);
            }
        }
        cout << ans.top(); ans.pop();
        while (!ans.empty())
        {
            cout << ' ' << ans.top();
            ans.pop();
        }
        cout << endl;
    }
    return 0;
}
```
