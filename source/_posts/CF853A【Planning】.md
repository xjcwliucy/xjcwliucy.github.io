---
title: CF853A【Planning】
date: 2023-03-09 13:31:57
category: ['题解']
# banner: 
---
正解：**贪心**+**并查集**维护。

# Sulution of CF853A(Planning)

### 简述题意 / General describe:

题目要求将 $1,2,...,n$ 重新排列在 $k+1,k+2,...,k+n$ 的区间内，并产生 $\sum _{i=1}^{n} [(total_i-start_i)\times c_i]$ 的贡献。其中 $start_i$ 表示初始数值， $total_i$ 表示排列后数值， $c_i$ 为参数，由题目给出。

求一种排列方法，产生最小贡献。

### 心路历程 / General ideas:

1. 首先，**一眼贪心**。详细证明见下：

设贡献为 $w$：

 $w=\sum _{i=1}^{n} (total_i-start_i)\times c_i=\sum _{i=1}^{n}(total_i\times c_i-start_i\times c_i)$ 
 
 显然， $\sum_{i=1}^{n}(-start_i\times c_i)$ 为定值，只需构造 $\sum_{i=1}^n(total_i\times c_i)$ 尽可能小即可。
 
2. 思考贪心策略。 $c_i$ 为参数， $total_i$ **可以改变但不能重复**，那么对于更大的 $c_i$， $total_i$ 应当更小，即更靠近 $start_i$ ，证明只需任意反例做差即可，略。

3. 很快可以想到，对 $c_1,c_2,...,c_n$ 从大到小排序，并依次为他们找到目前可用的最小 $total$ 计算贡献并记录即可。但是显然复杂度是 $O(n\log n+n^2)$，很劣。

4. 第一时间想优化掉遍历产生的 $O(n)$ 复杂度，因为每确定一个 $total$，其他位置的最佳选择都要随之变动，所以第一时间想到**并查集**，因为它可以实时改变状态。

5. 明白一点说，使用 $nr$ 数组记录当前每个 $start$ 的最优 $total$，每改变一个值，将该 $nr_{start}$ 修改为 $nr_{start}+1$，其他与之重复的值会随之改变。详见Code。

## 代码 / Code：

[AC记录](https://www.luogu.com.cn/record/103929898)

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>
using namespace std;

long long n,k;
long long out[(long long)3e5+5]; 
struct cost {
	long long time, value;
}c[(long long)3e5+5];
bool cmp(cost a, cost b) {
	return a.value > b.value;
}
long long ans;
long long nr[(long long)6e5+5];
long long find(long long x) {
	if(nr[x] == x)
		return x;
	return nr[x] = find(nr[x]);
}

int main() {
	freopen("plan.in","r",stdin);
	freopen("plan.out","w",stdout);
	scanf("%lld%lld", &n, &k);
	for(long long i = 1 ; i<= n ; i++) {
		scanf("%lld", &c[i].value);
		c[i].time = i;
	}
	for(long long i = 1 ; i<= k ; i++) {
		nr[i] = k+1;
	}
	for(long long i= k+1 ; i<= k+n ; i++) {
		nr[i] = i;
	}
	sort(c+1, c+n+1, cmp);
	for(long long i = 1 ; i<= n ; i++) {
		long long place = c[i].time;
		ans += c[i].value*(find(place)-place);
		out[place] = find(place);
		nr[find(place)] = find(place)+1;
	}
	printf("%lld\n",ans);
	for(int i = 1 ; i<= n ; i++) {
		printf("%lld ", out[i]);
	}
	fclose(stdin);
	fclose(stdout);
	return 0;
} 
```

以上即是考场代码，希望有所帮助。
