---
title: "백준 Gold 4 네트워크 연결"
date : "2025-09-06 11:00:00 +0900"
last_modified_at: "2025-09-06T11:00:00"
categories:
  - 코딩 테스트
tags:
  - 그래프
  - MST
  - 최소 신장 트리
---

## 네트워크 연결 (백준 Gold 4)
<https://www.acmicpc.net/problem/1922><br>

n개의 '컴퓨터'를 연결하는데<br>
m개의 연결할 수 있는 선의 수가 주어진다<br>

그리고 m개의 간선 정보가 제공된다<br>
시작점 - 끝점 - 선의 비용<br>

모든 컴퓨터를 연결하면서<br>
동시에 '가능한 적은 비용'을 들게 하고 싶을때의<br>
최소 비용 구하기 문제<br>

## 풀이 방법
'모든 노드'가 연결되는 최소 비용을 구하는 문제<br>
(최소 스패닝 트리 (최소 신장 트리) - MST)<br>

크게 2가지 방식으로 풀 수 있는데<br>

- 크루스칼<br>
  : 주어지는 전체 간선을 '비용'순으로 정렬하고<br>
    값이 '저렴한 순서'대로 간선을 연결한다<br>
	다만 '이미 연결된' 간선이라면 pass<br>
	(사이클 체크)<br>
	(이걸 확인하기 위하여 보통 union-find 자료구조를 이용)<br>

- 프림<br>
  : 임의의 시작 노드를 정하고<br>
    그 노드의 모든 간선을 비용을 기준으로 한 '최소 힙'에 넣어준다<br>
	이후 비용이 적은 간선을 이용하여 '다음 방문 노드 체크'<br>
	(이미 방문했다면 pass)<br>
	다음 방문한 노드의 간선도 전부 힙에 넣어준다<br>
	전부 방문 완료하였다면 비용 반환<br>

개인적으론 프림 알고리즘을 더 선호하는 편이기에<br>
해당 방식으로 구현하였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
#include<unordered_set>
#include<algorithm>
#include<limits.h>
#include<queue>

using namespace std;

typedef pair<int, int> pii;

struct Compare
{
	bool operator()(const pii& a, const pii& b)
	{
		return a.second > b.second;
	}
};

int prim(unordered_map<int, vector<pii>>& maps, int start, int n)
{
	int result = 0;

	priority_queue<pii, vector<pii>, Compare> pq;

	unordered_set<int> visit;
	visit.insert(start);

	for (auto& p : maps[start])
	{
		pq.push(p);
	}

	while (pq.empty() == false)
	{
		int next = pq.top().first;
		int nextCost = pq.top().second;
		pq.pop();

		if (visit.find(next) != visit.end())
			continue;

		visit.insert(next);
		result += nextCost;

		if (visit.size() == n)
			break;

		for (auto& p : maps[next])
			pq.push(p);
	}

	return result;
}

int main()
{
	int n, m;
	cin >> n >> m;

	unordered_map<int, vector<pii>> maps;

	int st = 0, startCost = INT_MAX;

	for (int i = 0; i < m; i++)
	{
		int start, to, cost;
		cin >> start >> to >> cost;

		maps[start].push_back({ to,cost });
		maps[to].push_back({ start,cost });

		if (cost < startCost)
		{
			startCost = cost;
			st = start;
		}
	}

	cout << prim(maps, st,n);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/8b78279f-b99d-4b4f-9952-75f8f44bd34d)](https://github.com/user-attachments/assets/8b78279f-b99d-4b4f-9952-75f8f44bd34d){: .image-popup}<br>

프림 알고리즘은 다익스트라 구현에 익숙하다면<br>
금방 익힐 수 있는 알고리즘이기에<br>
더 친숙한 듯 하다<br>

오랜만에 푸는 mst 계열 문제였으나<br>
다행히 잘 풀 수 있었다<br>