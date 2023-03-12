---
title: 最短路算法集合【A collection of shortest path algorithms】
date: 2023-03-12 12:26:01
category: ['算法']
---

# 单源最短路算法

## 一、迪杰斯特拉算法【Dijkstra】

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

持续更新中。。。。。。