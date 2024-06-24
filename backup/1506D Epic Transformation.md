## 原题链接
[Epic Transformation](https://codeforces.com/contest/1506/problem/D)

## 题意简述
给定正整数序列，每次操作删除两个不同的数字，问任意次操作后序列中最少剩余多少个数。

## 思路
统计每个数字出现的次数，并对出现次数由小到大排序，排序后的数组记为：

$$a_1 \leq a_2 \leq  ... \leq a_m$$
$$\sum_{i=1}^{m} a_i = n$$
我们首先考察最大的数字 $a_m$：

1. 若 $a_m > \frac{n}{2}$，则用 $a_m$ 中的数消去其他的数，最后剩下 $a_m - (n - a_m)$ 个数

2. 若 $a_m \leq \frac{n}{2}$，则将原序列中出现次数最多的两个数各消去一个，得到：
$$a_1 \leq a_2 \leq  ... \leq a_{m-2}$$
$$a_{m-1}-1 \leq a_{m}-1$$
我们对上面的 $\{a_i\} $ 重新从小到大排序得到 $\{b_i\}$，下面证明 $\{b_i\}$ 中的最大值依然小于等于 $\frac{\sum b_i}{2}$:

一方面，
$$a_{m-1}-1 \leq a_{m}-1 \leq \frac{n-2}{2} = \sum b_i$$

另一方面，若 $a_{m-2} \geq \frac{n}{2}$ 则 $a_{m-2}+a_{m-1}+a_m \geq \frac{3n}{2}$ 矛盾
从而
$$a_{m-2} < \frac{n}{2}$$
$$\Rightarrow a_{m-2} \leq \frac{n-2}{2} = \sum b_i $$
故
$$\max \{b_i\} = \max \{a_i\} = \max \{a_{m-2},a_{m-1}-1, a_m-1\} \leq \sum b_i$$

这样一来，我们把排序后的 $\{b_i\}$ 看成原来的 $\{a_i\}$ 然后递归下去，数组的总和不断减小，直至数组为空或只剩下1。

因此，本题最终答案为 $\max \{a_m - (n - a_m), n\mod 2\}$