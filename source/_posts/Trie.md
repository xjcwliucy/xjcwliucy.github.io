---
title: 字典树笔记【Trie】
date: 2023-03-19 13:22:36
category: ['算法']
---

## 算法简介：

字典树，又称单词查找树，Trie 树，是一种树形结构，能够在 $O(n\cdot len)$ 的时间复杂度内查询 $n$ 个匹配串是否在模式串中出现。作为例子，请看例题：[字典树模板](https://www.luogu.com.cn/problem/P2580)

下面我会告诉大家如何实现它。

## 算法实现：

个人喜欢使用指针版本，所以不讲解非指针版本。

1. 首先我们考虑如何定义该树的其中一个结点。对于一个结点，它应该有：

- 指向 26 个字母的指针，它们指向该结点的 26 个儿子（如果它有的话）。
- 一个标记，记录该结点是否为一个单词的结尾。

代码化的，它应该被这样定义：

```cpp
const int LetterID = 26;
int mapping(char ch){return ch-'a';}

struct Node
{
	Node *Son[LetterID];
	int Value;
	Node()//构造函数，当我们为它申请空间时，会自动调用这个函数，完成初始化。
	{
		Value = 0;
		for(int i = 0 ; i< LetterID ; i++)
			Son[i] = NULL;
	}
};

```

2. 其次，一棵 Trie 应该可以插入一些字符串，作为模式串。具体操作应该如下：

- 步骤一：获取当前枚举到的模式串中的字符，检查当前结点是否有这个儿子，如果有，跳转步骤二；否则，跳转步骤三。
- 步骤二：将记录当前在树上的位置的指针移动到该儿子。如果字符串结束，跳转步骤四；否则，遍历下一个字符，跳转步骤一。
- 步骤三：申请一个指向 Node 类型的指针，并作为当前结点的该字符儿子，并将记录当前在树上的位置的指针移动到该儿子。如果字符串结束，跳转步骤四；否则，遍历下一个字符，跳转步骤一。
- 步骤四：将当前结点的标记设为 1，表示这里是一个模式串的结尾，这里有一个正确的单词。

代码化的，它应当是个这样的函数：
```cpp
int mapping(char ch){return ch-'a';}

void Insert(Node* Root, string s)
{
	Node* Now = Root;// Root 是指向整个 Trie 树的根结点的指针。
	for(int i = 0 ; i< s.size() ; i++)
	{
		if(Now->Son[mapping(s[i])] == NULL)
			Now->Son[mapping(s[i])] = new Node;
		Now = Now->Son[mapping(s[i])]; 
	}
	Now->Value = 1;
	return ;
}
```

3. 最后，一棵 Trie 树应当有查询功能。即查询某个单词是否在之前的模式串中出现过。这个阶段应当不产生任何新结点。

- 步骤一：获取当前枚举到的模式串中的字符，检查当前结点是否有这个儿子，如果有，跳转步骤二；否则，返回 false，表示没有这个单词。
- 步骤二：将记录当前在树上的位置的指针移动到该儿子。如果字符串结束，返回 true，表示查找到了；否则，遍历到下一个字符，跳转步骤一。

代码化的，这个函数应当是这个样子：
```cpp
bool Find(Node* Root, string s)
{
	Node* Now = Root;
	for(int i = 0 ; i< s.size() ; i++)
	{
		if(Now->Son[mapping(s[i])] == NULL)
			return false;
		Now = Now->Son[mapping(s[i])];
	}
	return Now->Value == 1 ? true : false;
}

```

4. 最后，上题代码如下：
```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <map>
using namespace std;
const int LetterID = 26;
int mapping(char ch){return ch-'a';}

struct Node
{
	Node* Son[LetterID];
	int Value;
	Node()
	{
		Value = 0;
		for(int i = 0 ; i< LetterID ; i++)
			Son[i] = NULL;
	}
};

int N;
string S;
map<string, string> Mp;/* map<Name, Status> Mp */

void Insert(Node* Root, string s)
{
	Node* Now = Root;
	for(int i = 0 ; i< s.size() ; i++)
	{
		if(Now->Son[mapping(s[i])] == NULL)
			Now->Son[mapping(s[i])] = new Node;
		Now = Now->Son[mapping(s[i])]; 
	}
	Now->Value = 1;
	return ;
}

bool Find(Node* Root, string s)
{
	Node* Now = Root;
	for(int i = 0 ; i< s.size() ; i++)
	{
		if(Now->Son[mapping(s[i])] == NULL)
			return false;
		Now = Now->Son[mapping(s[i])];
	}
	return Now->Value == 1 ? true : false;
}

Node* Root = new Node;

int main()
{
	scanf("%d", &N);
	for(int i = 1 ; i<= N ; i++)
	{
		cin >> S;
		Insert(Root, S);
	}
	scanf("%d", &N);
	for(int i = 1 ; i<= N ; i++)
	{
		cin >> S;
		if(Mp.find(S) != Mp.end())
			if(Mp[S] == "OK")
				printf("REPEAT\n");
			else
				printf("WRONG\n");
		else
		{
			if(Find(Root, S) == true)
				printf("OK\n"), Mp.insert({S,"OK"});
			else
				printf("WRONG\n"), Mp.insert({S,"WRONG"});
		}
	}
	return 0;
}
```

# End