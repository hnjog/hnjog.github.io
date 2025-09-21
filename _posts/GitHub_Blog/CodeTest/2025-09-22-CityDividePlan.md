---
title: "백준 Gold 4 도시 분할 계획"
date : "2025-09-22 10:30:00 +0900"
last_modified_at: "2025-09-22T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - 최소 스패닝 트리
  - MST
  - 크루스칼
  - Union Find
---

## 도시 분할 계획 (백준 Gold 4)
<https://www.acmicpc.net/problem/1647><br>

N개의 집과 M개의 길이 주어지고<br>
이러한 마을을 2개로 분할하면서<br>
동시에 유지비를 최소로 하기 위한 길들만 남겨두려 할때<br>
조건에 맞는 최소 유지비를 구하는 문제<br>

## 풀이 방법

최소 스패닝 트리(MST,최소 신장 트리)를 구하는 문제<br>
요점은 마을을 2개로 나눈 후의 **'최소 유지비'**를 구하는 것이다<br>

따라서 MST 비용을 구한 이후<br>
`가장 비용이 큰 간선의 비용을 제거`하게 되면<br>
자연스럽게 마을은 둘로 나뉘게 되고<br>
문제의 해답을 구할 수 있게 된다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

using namespace std;

int FindParent(vector<int>& pv, int x)
{
	if (pv[x] == x)
		return x;

	return pv[x] = FindParent(pv, pv[x]);
}

bool Union(vector<int>& pv, int a, int b)
{
	a = FindParent(pv, a);
	b = FindParent(pv, b);

	if (a == b)
		return false;

	pv[a] = b;

	return true;
}

struct edge
{
	int s, t, cost;
};

int main()
{
	int n, m;
	cin >> n >> m;

	vector<edge> edges;
	vector<int> parents(n+1);

	for (int i = 0; i < n + 1; i++)
	{
		parents[i] = i;
	}

	for (int i = 0; i < m; i++)
	{
		edge e;
		cin >> e.s;
		cin >> e.t;
		cin >> e.cost;

		edges.push_back(e);
	}

	sort(edges.begin(), edges.end(), []
	(const edge& a, const edge& b)
		{
			return a.cost < b.cost;
		});

	long answer = 0;
	long bestCost = 0;

	for (edge& e : edges)
	{
		if (Union(parents, e.s, e.t))
		{
			answer += e.cost;
			if (bestCost < e.cost)
				bestCost = e.cost;
		}
	}

	cout << answer - bestCost;
	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/f06362c6-079e-4505-8457-e6c4953614e6)](https://github.com/user-attachments/assets/f06362c6-079e-4505-8457-e6c4953614e6){: .image-popup}<br>

이전에는 Prim 알고리즘의 DFS 방식이 더 익숙하였으나<br>
막상 `Union Find`의 코드 스니펫을 외우니<br>
이 쪽이 더 편한것 같기도 하다<br>