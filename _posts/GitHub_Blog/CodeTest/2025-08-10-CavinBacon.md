---
title: "백준 Silver 1 케빈 베이컨의 6단계 법칙"
last_modified_at: "2025-08-10T07:00:00"
categories:
  - 코딩 테스트
tags:
  - BFS
  - 그래프
---

## 케빈 베이컨의 6단계 법칙  (백준 Silver 1)
<https://www.acmicpc.net/problem/1389><br>

BFS를 이용하여 풀 수 있는 문제이다<br>
각각의 사람들의 베이컨 수치를<br>
BFS를 통해 구한 후,<br>
이후 마지막에 그 값을 비교하여<br>
최수 수치인 사람의 index를 출력한다<br>

베이컨 수치는<br>
각 요소에서 모든 다른 요소들까지의 거리의 합<br>

### BFS를 이용한 풀이 방식?
dfs가 재귀라면<br>
bfs는 queue를 통해 '깊이'보다 현재 자신의<br>
자식들을 우선하여 탐색한다<br>

```
DFS는 재귀를 통해
'자식'의 자식으로 깊이 들어가서
결과를 찾는다

dfs
a - b - c - D
		  - E
	  - F
  - g

BFS는 Queue 등을 통해(Queue가 제일 구현하기 편리하다)

a - b
a - g
a - b - c
a - b - f
a - b - c - d
a - b - c - e

그렇기에 대부분의 경우의
최단 경로 탐색에는 bfs를 이용하는 것이
유리한 편
```

### 결과

<img width="1171" height="113" alt="Image" src="https://github.com/user-attachments/assets/dfcaeffd-6b0f-4bff-b792-6808604a6a0a" /><br>

다행히 bfs 자체는 여러번 구현해본 적이 있기에<br>
구현 방식이 머리에 남아 있었다<br>
어렵지 않게 풀 수 있었다<br>

### 제출 코드

```
#include<iostream>
#include<unordered_map>
#include<vector>
#include<queue>
#include<limits.h>

using namespace std;

int n, m;

int bacon(unordered_map<int, vector<int>>& pfmap, int people)
{
	int sums = 0;

	struct fInfo
	{
		int now;
		int nowCost;
	};

	queue<fInfo> q;
	q.push({ people,0 });

	vector<int> v(n + 1);

	while (q.empty() == false)
	{
		int now = q.front().now;
		int nowCost = q.front().nowCost;
		q.pop();

		if (v[now] != 0)
			continue;

		v[now] = nowCost;

		for (int next : pfmap[now])
		{
			q.push({ next,nowCost + 1 });
		}
	}

	for (int i : v)
	{
		sums += i;
	}

	return sums;
}

int main()
{
	cin >> n >> m;

	unordered_map<int, vector<int>> pfmap;
	vector<int> vec(n+1,0);

	for (int i = 0; i < m; i++)
	{
		int t1, t2;
		cin >> t1 >> t2;

		pfmap[t1].push_back(t2);
		pfmap[t2].push_back(t1);
	}

	for (int i = 1; i <= n; i++)
	{
		vec[i] = bacon(pfmap, i);
	}

	int result = -1;
	int resultCost = INT_MAX;

	for (int i = 1; i <= n; i++)
	{
		if (vec[i] < resultCost)
		{
			result = i;
			resultCost = vec[i];
		}
	}

	cout << result;

	return 0;
}
```