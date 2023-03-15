---
title: 最短路算法集合【A collection of shortest path algorithms】
date: 2023-03-12 12:26:01
category: ['算法']
---

# 单源最短路算法

介绍：此类最短路算法需要指定一个“源”点，并计算出该点到图中所有点的最短距离。

## 迪杰斯特拉算法【Dijkstra】

1. 算法简介：迪杰斯特拉【Dijkstra】是由 $Edsger Wybe Dijkstra$ 发明的**单源**最短路算法，其朴素版本可以在 $O(n^2)$ 的时间复杂度内求解出连通图内所有点到一个定点的最短路线。
2. 算法思想：该算法采用**贪心**的策略，从“源”开始作为基准点，更新与其相邻的点到“源”的距离，并选其中**距离“源”最近的、没有作为过基准点**的点作为新的基准点，进行下一次遍历，直到图中所有点都成为过基准点。
3. 例题（[Luogu-P3371【单源最短路弱化版】](https://www.luogu.com.cn/problem/P3371)） & 算法实现：

```cpp
#include <iostream>
#include <cstdio> 
#include <map>
#include <bitset>
#include <vector>
#define INF 2147483647
using namespace std;

long long N,M,Root;
long long Dis[100005];
bitset<100005> Vist;
vector<long long> edge[100005];
map<long long,long long> Mp[100005];
//本人习惯于使用vector存边的关系；使用map存边的长度

void Dijkstra(long long Root)//Root => “源”
{
	Vist.reset();//bool数组，空间大小 O(n)，用以记录每个节点是否已经成为过基准点
	for(long long i = 1 ; i<= N ; i++)
		Dis[i] = INF;//Dis数组，记录每个点到“源”的距离
	Dis[Root] = 0;
	for(long long i = 1 ; i<= N ; i++)//每个点都应成为一次基准点，故循环 n 次
	{
		long long Min = INF;
		long long K = 1;
        //Min => 当前轮Dis的最小值；K => 该最小值对应的节点
        //请注意：此处为不严谨写法，因为 K 节点的 Dis值不一定为 INF，但下一处循环一定会更新到它，所以不必担心
		for(long long j = 1 ; j<= N ; j++)
		{
			if(!Vist[j] && Dis[j] < Min)
			{
				Min = Dis[j];
				K = j;
			}
		}
		Vist[K] = true;//K 成为基准点
		for(vector<long long>::iterator it = edge[K].begin() ; it != edge[K].end() ; it++)
		{
			if(!Vist[*it] && Min+Mp[K][*it] < Dis[*it])
			{
				Dis[*it] = Min+Mp[K][*it];//更新与 K 相邻的点的 Dis值
			}
		}
	}
	for(long long i = 1 ; i<= N ; i++)
		printf("%lld ",Dis[i]);
}
int main()
{
	scanf("%lld%lld%lld",&N,&M,&Root);
	for(long long i = 1,L1,L2,Value ; i<= M ; i++)
	{
		scanf("%lld%lld%lld",&L1,&L2,&Value);
		edge[L1].push_back(L2);
		if(Mp[L1].find(L2) == Mp[L1].end())//判断重边，若有，取最小值边
			Mp[L1].insert({L2,Value});
		else
			Mp[L1][L2] = min(Mp[L1][L2],Value);
	}
	Dijkstra(Root);
	return 0;
}
```
4. 迪杰斯特拉**优化**：对于一个单源最短路的算法而言， $O(n^2)$ 的时间复杂度绝对算不上优秀，于是，**比迪杰斯特拉算法本身更重要**的优化算法————**堆优化**，产生了。
5. 堆优化思路：详细阅读迪杰斯特拉算法的代码，会发现每一次求最小值时需要完全遍历一遍，直接贡献了 $O(n)$ 的时间复杂度，用脚想一想也知道是可以————至少是应该，优化的。这里，我们选用了**小根堆**的数据结构进行优化。小根堆是一种树形结构，可以使用 $O(\log n)$ 的时间插入，并用 $O(1)$ 的时间复杂度获取**所有已插入值中的最小值**，于是整体时间复杂度被降至 $O(n\log n)$，这样的复杂度适用于 大/中 型图，这意味着优化版的迪杰斯特拉是优秀的单源最短路算法。
6. 堆优化的实现：小根堆可以自行实现，也可以直接使用 c++ 的 stl 库中的 $priority_queue$（“优先队列”），为了不影响代码的整体结构，我通常更乐于使用 stl 的内容，这也是我推荐的做法。如果您乐于自行实现，可以自行百度做法。
7. 例题([Luogu-P4779【单源最短路标准版】](https://www.luogu.com.cn/problem/P4779)) & 算法实现：
```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <map>
#include <queue>
#include <bitset>
#include <cstring>
#define maxn 100005
using namespace std;

int N,M,S;
int Dis[maxn];
vector<int> Edge[maxn];
map<int, int> Mp[maxn];
priority_queue<pair<int,int>, vector<pair<int,int> >, greater<pair<int,int> > > Q;//定义小根堆
// 实际上，stl中也有大根堆，可以取出最大值，使用方法与小根堆别无二致，定义方法如下：
// priority_queue<type(int,char and so on), vector<type>, less<type> > name;
bitset<maxn> Vist;

void BetterDijkstra(int Root)
{
	while(!Q.empty()) Q.pop();
	Vist.reset();
	memset(Dis,0x3f,sizeof(Dis) );
	Dis[Root] = 0;
	Q.push({0,Root});
    // pair<int,int> 是一个复合类型，由两个 int 组成，使用 name.first 访问第一个int值，并用 name.second 访问第二个。
    // 当给 pair<int,int> 排序时，会自动以 name.first 的值作为唯一关键字进行排序。现在将 name.first 赋值为基准点的 Dis值，整个优先队列就会自动按照 Dis值升序排列。
	while(!Q.empty())
	{
		int V = Q.top().first;
		int Ben = Q.top().second;
        // Q.top() 即是预备基准点的信息
		Q.pop();
		if(Vist[Ben]) continue;
		Vist[Ben] = true;
		for(int End : Edge[Ben])// C++11及以后版本适用的vector遍历方法
		{
			if(Vist[End]) continue;
			Dis[End] = min(Dis[End], V+Mp[Ben][End]);
			Q.push({Dis[End], End});
		} 
	}
	return ;
}

void MakeEdge(int L1, int L2, int V)// 加边函数
{
	Edge[L1].push_back(L2);
	if(Mp[L1].find(L2) == Mp[L1].end())
		Mp[L1].insert({L2,V});
	else
		Mp[L1][L2] = min(Mp[L1][L2], V);
	return ;
}

int First, End, Value; 

int main()
{
	scanf("%d%d%d",&N, &M, &S);
	for(int i = 1 ; i<= M ; i++)
	{
		scanf("%d%d%d", &First, &End, &Value);
		MakeEdge(First,End,Value);	
	}
	BetterDijkstra(S);
	for(int i = 1 ; i<= N ; i++)
		printf("%d ",Dis[i]);
	puts("");
	return 0;
} 
```

## SPFA

1. 算法简介：$Bellman-Ford$ 算法的队列优化版本，与 $Bellman-Ford$ 有同样的功能，却有平均更优的时间复杂度，故这里不讲解 $Bellman-Ford$，仅讲解 $SPFA$。
2. 算法功能：可求单源最短路，并可以解决**带负边权和带负环**的图，或者换句话说，就是**迪杰斯特拉的负边权适配版本**。因为要适配负边权，所以**比迪杰斯特拉有更劣的时间复杂度**，最慢为 $O(VE)$。
3. 算法思想：与迪杰斯特拉非常相像，因为迪杰斯特拉只用处理正边权，所以一个点只需要成为一次基准点就可以了，因为目前所有的点的 Dis值都已经大于该点，之后也不可能拐个弯就更小了；SPFA 不一样，因为有负边权，所以是真的有可能“拐个弯反而比该点小”的，此时就需要**让该点再一次成为基准点**，进行更新。所以相较于迪杰斯特拉，只需要**在一个点出队后把它的基准点标记删除**，使其可以再次成为基准点，该算法就变成了SPFA。
4. 双端队列优化：在这里有一个极小极小（指码量）的优化，但是可以大大提升算法的时间复杂度。因为SPFA算法需要多次更新，所以给每次更新排序成为了没有意义的操作。这意味着迪杰斯特拉中的优先队列优化对SPFA没有意义，但是我们仍有一种情况需要尽量获得更小的基准值——判断负环。因为一旦找到负环我们就应该中止程序，所以越早找到负环就可以越早完成；但是毫无疑问，为了一个不确定的复杂度而在所有的情况下加上 $O(\log n)$ 的复杂度是很不划算的，所以我们可以获得一个近似的优化——**将即将插入队列的值与队首做比较，如果比队首的小，那么插入队首；否则，插入队尾**。这样的操作在一定程度上获得了局部有序。
5. 判断负环：相较于迪杰斯特拉，SPFA在处理负环方面格外有效。我们可以思考，如果一个图中没有负环，一个点最多被更新多少次？实际上，**对于一个有 $n$ 个点的图，一个点最多有 $n-1$ 条连边，每一条连边最多更新它一次**。那么相反的，**只要我们发现一个点被更新超过了 $n-1$ 次，那么就可以说明图中有至少一个负环**。
6. 例题（[Luogu-P2850 [USACO06DEC]Wormholes G](https://www.luogu.com.cn/problem/P2850)）& 代码示例：
```cpp
#include <cstdio>
#include <iostream>
#include <vector>
#include <map>
#include <queue>
#include <bitset>
using namespace std;
const int INF = 0x3f3f3f3f;
const int maxn = 502;

int t;
int n,m,w;
int ans;
int cnt[maxn];
vector<int> edge[maxn];
map<int, int> mp[maxn];
bitset<maxn> vist;
int dis[maxn];
deque<int> q;
int u, v, p;
void mem();
void make_edge(const int &L1, const int &L2, const int &V);
bool SPFA(int root);

int main() {
	scanf("%d", &t);
	while(t--) {
		mem();
		scanf("%d%d%d", &n, &m, &w);
		for(int i = 1 ; i<= m ; i++) {
			scanf("%d%d%d", &u, &v, &p);
			make_edge(u, v, p);
			make_edge(v, u, p);
		}
		for(int i = 1 ; i<= w ; i++) {
			scanf("%d%d%d", &u, &v, &p);
			make_edge(u, v, -p);
		}
		if(SPFA(1))
			puts("YES");
		else
			puts("NO");
	}
	return 0;
} 

void mem() {
	ans = INF;
	for(int i = 0 ; i< maxn ; i++)
		edge[i].clear();
	for(int i = 0 ; i< maxn ; i++)
		mp[i].clear();
	for(int i = 0 ; i< maxn ; i++)
		cnt[i] = 0;
	return ;
}

void make_edge(const int &L1, const int &L2, const int &V) {
	edge[L1].push_back(L2);
	if(mp[L1].find(L2) == mp[L1].end()) {
		mp[L1].insert({L2,V});
	} else {
		mp[L1][L2] = min(mp[L1][L2], V);
	}
	return ;
}

bool SPFA(int root) {
	vist.reset();
	for(int i = 1 ; i<= n ; i++) {
		dis[i] = INF;
	}
	dis[root] = 0;
	q.clear();
	q.push_back(root);
	vist[root] = true;
	while(!q.empty()) {
		int ben = q.front();
		q.pop_front();
//		cout << ben << endl;
		for(int tmp : edge[ben]) {
			if(dis[tmp] > dis[ben]+mp[ben][tmp]) {
				dis[tmp] = min(dis[tmp], dis[ben]+mp[ben][tmp]);
				cnt[tmp]++;
				if(cnt[tmp] > n-1)
					return true;
				if(!q.empty() && dis[tmp] <= dis[q.front()] && !vist[tmp]) {
					q.push_front(tmp);
					vist[tmp] = true;
				} else if(!vist[tmp]) {
					q.push_back(tmp);
					vist[tmp] = true;
				}
			}
			
		}
		vist[ben] = false;
	}
	return false;
}
```
思路解释：原题意要求回答是否可以从一个点回到该点并带回负权值，这显然就是负环的定义，只需要使用SPFA+双端队列优化判断负环即可。

# 多源最短路

介绍：此类算法不需要指定一个“源”点，Ta可以计算出图中任意两个点之间的距离。

## 弗洛伊德算法【Floyd】

1. 算法简介： $Floyd$ 是一种多源最短路算法，能够在 $O(n^3)$ 的时间里计算出任意两个点之间的距离。该算法采用动态规划的思路。
2. 算法功能：正负权对 $Floyd$ 没有影响，Ta可以处理绝大部分情况下的多元最短路问题。
3. 算法思路：首先，我们当然应该枚举 $i$ 点和 $j$ 点，作为此轮要更新的边的左右节点。其次，该算法要求我们枚举一个额外的变量 $k$，并比较 $i$ 到 $j$ 的距离和 $i$ 先到 $k$，再从 $k$ 到 $j$ 的距离，如果后者小，更新边权。实际上，此时 $Dis[i][j]$ 代表的意义是：只经过 $1,2,\dots,k$ 节点时， $i$ 到 $j$ 的最短距离。
4. 注意事项：因为 $Floyd$ 采用动态规划的思想，所以**保证无后效性**是无比重要的。所以当我们安排三层循环的位置时，必须把 $k$ 层安排在最外面。可以参照 $Dis[i][j]$ 的意义来理解这一点。
5. 例题（[P2910 [USACO08OPEN]Clear And Present Danger S](https://www.luogu.com.cn/problem/P2910)）& 示例代码：
```cpp
#include<stdio.h>
#include<valarray>
int a[101][101] , dis[100001];
int n , m ;
int main()
{
	scanf("%d%d" , &n , &m );
	for(int i = 1 ; i <= m ; i++) {
		scanf("%d" , &dis[i]);	
	}
	for(int i = 1 ;  i<= n ; i++) {
		for(int j = 1 ; j <= n ; j++) {
			scanf("%d" , &a[i][j]);
		}
	}
// Floyd 核心代码
	for(int k = 1 ; k<= n ; k++) {
		for(int i = 1 ; i<= n ; i++) {
			for(int j = 1 ; j<= n ; j++ ) {
				a[i][j] = std::min(a[i][j] , a[k][j] + a[i][k]);
			}
		}
	}
	long long ans = 0;
	for(int i = 1 ; i< m ; i++) {
		ans += a[dis[i]][dis[i+1]];
	}
	printf("%lld" , ans);
	return 0;
}
```
本来还有 Johnson全源最短路算法，但是我懒，且其不必须，故不更新。希望了解->自行查阅。