---
title: 题解 [atcoder_abc_172_e] NEQ
date: 2023-04-25 09:33:55
category: ['题解']
# banner:
---

被这题考场 $\text{AK}$ 了，教练请了数学竞赛老师来讲，赛后一遍 $\text{AC}$ 了，于是写篇题解纪念一下。

# 前置芝士：

**请确保您掌握集合的基础知识。**

定义 $|A|$ 表示集合 $A$ 的大小，$|B|$ 表示集合 $B$ 的大小，以此类推。
则有：
$$|A\cup B|=|A|+|B|-|A\cap B|$$ 
如果您感到困难，画个图理解一下。我们继续：
$$|A\cup B\cup C|=(|A|+|B|+|C|)-(|A\cap B|+|B\cap C|+|A\cap C|)+(|A\cap B\cap C|)$$ 
这次更建议您画图理解。接下来经过**科学论证法**，我们可以获得**推广公式**：
$$|A_1\cup A_2\cup \dots \cup A_n|=(\sum^{n}_{i=1}|A_i|)-(\sum^{n}_{i=1}\sum^{n}_{j=1 ,j\neq i}|A_i\cap A_j|)+\dots \pm|A_1\cap A_2\cap\dots \cap A_n|$$ 
最后一个 $\pm$ 符号视 $n$ 奇偶性而定，$n$ 为奇数则加；$n$ 为偶数则减。

**请确保您可以理解**。我们开始分析题目。

# 题目描述：

对于两个大小为 $n$ 集合 $E,F$，需要满足：

- 条件一：$\forall i\in [1,n],E_i\neq F_i$；

- 条件二：$\forall i\in [1,n],E_i\in [1,m]$；

- 条件三：$\forall i\in [1,n],F_i\in [1,m]$。

求可能的方案。

# 题目分析：

首先我们确定 $E$ 集合，毫无疑问可以获得 $\text{A}^{n}_{m}$ 个方案。（此处 $\text{A}^{k_1}_{k_2}$ 是排列组合的用法，表示从 $k_2$ 个元素中选择 $k_1$ 个元素并考虑顺序的方案数。）

接下来我们尝试确定 $F$ 集合。如果不考虑条件一，那么显然也有 $\text{A}^{n}_{m}$ 个方案，与 $E$ 集合的相同。在此基础上考虑条件一，我们显然要**剔除一些方案**。 参见下图：

![](https://cdn.luogu.com.cn/upload/image_hosting/tr282inv.png)

该图列举出了**所有的不合法方案**，对于集合 $T_i$，它的元素是**一些排列**，是不同的 $F$ 集合，不过需要满足 $F_i=E_i$。

值得强调的是，$E_i=F_i$ **并不意味着一定是 $E$ 集合的第 $i$ 个等于 $F$ 集合的第 $i$ 个，而是表示保证有一个位置上 $E$ 和 $F$ 相同。画出 $T_1\dots T_n$ 则强调对于每一个集合 $T_i$，该位置均不相同**。

**请务必理解这一点！**

接下来我们继续：

那么是不是直接用 $\text{A}^{n}_{m}$ 减去 $(|T_1|+|T_2|+\dots |T_n|)$ 就可以了呢？当然不是这样，因为 $T_1,T_2,\dots ,T_n$ 之间也有重复的部分，所以我们要减去的是 $|T_1\cup T_2\cup \dots \cup T_n|$，接下来，您果然熟练运用了我们的前置芝士，写出如下等式：

$$|T_1\cup T_2\cup \dots \cup T_n|=(\sum^{n}_{i=1}|T_i|)-(\sum^{n}_{i=1}\sum^{n}_{j=1 ,j\neq i}|T_i\cap T_j|)+\dots \pm|T_1\cap T_2\cap\dots \cap T_n|$$

最后，您甚至避免了歧义，说道：“**最后一个 $\pm$ 符号视 $n$ 奇偶性而定，$n$ 为奇数则加；$n$ 为偶数则减**。”

**感谢您做出的贡献**！但是接下来怎么办呢？

让我们回到最开始的图，会发现，尽管求 $|T_1\cup T_2\cup\dots\cup T_n|$ 是很困难的，但是求 $|T_1\cap T_2\cap\dots\cap T_n|$ 却要容易得多，因为**取集合间的交集可以保留单个集合的性质，即 $E_i=F_i$**。

详细地说，$k$ 个 $T$ 类集合的交集的大小即为：对于 $F$ 集合，固定住 $k$ 个数的方案数。

因为这 $k$ 个数位置不定，**所以我们用 $\text{C}^{k}_{n}$ 表示**，即从 $n$ 个位置中找到 $k$ 个位置的方案，这是排列组合的内容。

而除了这 $k$ 个位置，其余的数要求大小在 $[1,m]$ 之间且互不相同。

幸运的是，我们依然可以用排列组合的公式来表示**这 $(n-k)$ 个位置的方案数，即 $\text{A}^{n-k}_{m-k}$**。它表示从 $(m-k)$ 个数中找出 $(n-k)$ 个不同的数并考虑顺序的方案数。

最后我们把这两种方案数乘起来，就是 **$k$ 个 $T$ 类集合的交集的大小**：

$$\text{C}^{k}_{n} \cdot \text{A}^{n-k}_{m-k}$$

**真是巨大的进步**！接下来让我们看到您推出的公式，发现我们需要处理 $1$ 到 $n$ 个集合的交集，那么枚举 $k$ 并加起来！这次又是您更先整理出式子了：

$$|T_1\cup T_2\cup \dots \cup T_n|=\sum^{n}_{k=1}[\text{C}^{k}_{n} \cdot \text{A}^{n-k}_{m-k}\cdot (-1)^k]$$

**这是计算机可以计算出的式子了！**

我们再来看看它跟答案有什么关系吧！您一定还记得，我们是要用 $\text{A}^{n}_{m}$ 减去这个式子来求出 $F$ 的方案数吧，也就是说，$F$ 集合的方案数就是：

$$\text{A}^{n}_{m}-\sum^{n}_{k=1}[\text{C}^{k}_{n} \cdot \text{A}^{n-k}_{m-k}\cdot (-1)^k]$$

**离答案只有一步之遥了！**

您果然立刻就想起来我们还有 $\text{A}^{n}_{m}$ 个 $E$ 集合的可能方案数吧！最后的答案应该是 $E$ 集合的方案数乘上 $F$ 的方案数。您又抢先一步写出式子了:

$$\text{A}^{n}_{m}\times \{\text{A}^{n}_{m}-\sum^{n}_{k=1}[\text{C}^{k}_{n} \cdot \text{A}^{n-k}_{m-k}\cdot (-1)^k]\}$$

**没错，这就是答案了！**

快脱离复杂的数学运算，用这个简洁明快的式子去写出您的 $\text{AC}$ 代码吧！

我先放出我的代码啦！

```cpp

#include <iostream>
#include <cstdio>
#define MOD 1000000007
using namespace std;
typedef long long i64;

i64 fast_pow(i64 a, i64 p) {
	i64 r = 1;
	while(p) {
		if(p & 1) r = r*a%MOD;
		a = a*a%MOD;
		p >>= 1;
	}
	return r%MOD;
}
i64 n, m;
i64 j[500005], inv[500005];

i64 A(i64 a, i64 b) {
	return j[b]*inv[m-n]%MOD;
}
i64 C(i64 a, i64 b) {
	return j[b]*inv[a]%MOD*inv[b-a]%MOD;
}

int main() {
	scanf("%lld%lld", &n, &m);
	j[1] = j[0] = 1;
	for(i64 i = 2 ; i<= 500000 ; i++) {
		j[i] = i*j[i-1]%MOD;
	}
	inv[1] = inv[0] = 1;
	for(i64 i = 2 ; i<= 500000 ; i++) {
		inv[i] = inv[i-1]*fast_pow(i, MOD-2)%MOD;
	}
	i64 sum = 0;
	for(i64 i = 1 ; i<= n ; i++) {
		i64 v = C(i, n)*A(n-i, m-i)%MOD;
		if(i%2 == 1) v = (v*(-1)+MOD)%MOD;
		sum = (sum+v)%MOD;
	}
	sum = (sum+A(n, m))%MOD;
	sum = sum*(A(n, m))%MOD;
	printf("%lld\n", sum%MOD);
	return 0;
}

```

### 期待与您的下一次合作！祝您好运~
