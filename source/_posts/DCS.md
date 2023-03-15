---
title: 差分约束系统【Differential confinement system】
date: 2023-03-15 12:31:13
category: ['算法']
---

# 算法简介

**差分约束系统**（system of difference constraints），是求解关于一组变量的特殊不等式组的方法。形式化的，如果有一个包含 $n$ 个未知数、 $m$ 个不等式的不等式组形如：

$\begin{cases}X_{1}\leq x_{2}+v_{1}\\ x_{2}\leq x_{3}+v_{2}\\ \vdots \\ x_{m}\leq x_{m+1}+v_{m}\end{cases}$

那么我们就可以使用差分约束系统**求解出这个不等式组的一组解**。

# 算法原理

差分约束系统本质上是**将不等式组改编成 Dis值之间的关系**，通过**建图并跑最短路**来保证地图中的所有 Dis值都满足不等式要求。最终 Dis值就是不等式组的一组解。

举个例子：对于不等式 $x \le y+v$，Ta其实满足最短路图中的要求。回想我们更新 Dis值时的画面，大致如下：
```cpp
if(Dis[x] > Dis[y]+map[y][x]) {
    Dis[x] = Dis[y]+map[y][x];
}
```
不难发现，当 $Dis[x] > Dis[y]+map[y][x]$ 时，我们更新 Dis[x]，实际上就是要保证 $Dis[x] \le Dis[y]+map[y][x]$，而这恰好是不等式的要求。所以只要我们适当构造地图，就可以让所有未知数之间的关系成为一张图，并通过这样的转移去获得一组和谐的解。

更具体的，通过观察上式，我们可以让 map[y][x] 等于 v，来让转移代码完全等同于不等式。这意味着，**对于不等式 $x \le y+v$，我们应该建一条 y->x 且长度为 v 的有向边**。

# 算法实现

对于一个不等式组，Ta的一组解包含一些负数当然是不足为奇的，所以我们应该使用 SPFA 求解。

值得注意的是，如果图中出现负环，这说明不等式组无解。因为这代表着有形如 $a>b$ 且 $b>c$ 且 $c>a$ 的情况出现，而这当然是无法满足的。同时，我们无法排除图不连通的情况，所以我选择使用一个虚点“0”到所有点链接一条长度为 0 的点。

例题（[Luogu-P5960【模板】差分约束算法](https://www.luogu.com.cn/problem/P5960)）& Code：
```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>
#include <queue>
#include <map>
#include <bitset>
using namespace std;
const int maxn = 5003;
const int INF = 0x3f3f3f3f;

int n, m;
vector<int> edge[maxn];
map<int, int> mp[maxn];
bitset<maxn> vist;
deque<int> q;
int dis[maxn];
int cnt[maxn];
// 裸的SPFA
bool SPFA(int root) {
	for(int i = 0 ; i<= maxn ; i++) {
		dis[i] = INF;
		cnt[i] = 0;
	}
	while(!q.empty()) {
		q.pop_back();
	}
	vist.reset();
	q.push_back(root);
	dis[root] = 0;
	vist[root] = true;
	while(!q.empty()) {
		int ben = q.front();
		q.pop_front();
		for(int tmp : edge[ben]) {
			if(dis[tmp] > dis[ben]+mp[ben][tmp]) {
				dis[tmp] = dis[ben]+mp[ben][tmp];
				if(!vist[tmp]) {
					cnt[tmp] = cnt[ben]+1;
					if(cnt[tmp] > n) {
						return true;
					}
					if(dis[tmp] <= dis[q.front()]) {
						q.push_front(tmp);
					} else {
						q.push_back(tmp);
					}
					vist[tmp] = true;
				}
			}
		}
		vist[ben] = false;
	}
	return false;
}

void make_edge(int L1, int L2, int V) {
	edge[L1].push_back(L2);
	if(mp[L1][L2] == 0) {
		mp[L1][L2] = V;
	} else {
		mp[L1][L2] = min(mp[L1][L2], V);
	}
	return ;
}

int first, second, value;

int main() {
	scanf("%d%d", &n, &m);
	for(int i = 1 ; i<= m ; i++) {
		scanf("%d%d%d", &first, &second, &value);
		make_edge(second, first, value);
	}
	for(int i = 1 ; i<= n ; i++) {
		make_edge(0, i, 0);// 建立虚点
	}
	if(SPFA(0)) {
		puts("NO");
	} else {
		for(int i = 1 ; i<= n ; i++) {
			printf("%d ",dis[i]);
		}
	}
	return 0;
}
```

# 其他练习：

[Luogu-P1993 小 K 的农场](https://www.luogu.com.cn/problem/P1993)

[Luogu-P3275 [SCOI2011]糖果](https://www.luogu.com.cn/problem/P3275)

[Luogu-P7624 [AHOI2021初中组] 地铁](https://www.luogu.com.cn/problem/P7624)

[Luogu-P2294 [HNOI2005]狡猾的商人](https://www.luogu.com.cn/problem/P2294)