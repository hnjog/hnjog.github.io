---
title: "백준 Gold 3 최소비용 구하기 2"
date : "2025-10-23 10:30:00 +0900"
last_modified_at: "2025-10-23T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 다익스트라
  - 역추적
---

## 최소비용 구하기 2 (백준 Gold 3)
<https://www.acmicpc.net/problem/11779><br>

n개의 도시에서 단반향 버스 노선이 m개 주어질때<br>
시작 도시 에서 목표 도시까지 가는<br>
최소 비용과 그 경우의 방문 도시의 개수 와 경로를 출력하는 문제<br>

## 풀이 방법
최단 비용 거리를 구하는 동시에<br>
개수와 경로가 필요한 문제이다<br>

- 비용이 존재하기에 일반적인 BFS로는 풀 수 없음<br>
  (목적지가 바로 가는 버스가 있다면 비용을 고려하지 않으므로)<br>

- 다익스트라 알고리즘을 사용<br>
  (Priority_queue)

- 최소 비용을 기준으로 맵을 순회하며<br>
  경로 값을 갱신<br>
  (시작 비용은 0)<br>

- 다만 '경로' 또한 계산해야 하므로<br>
  visit를 pair<int,int>로 체크하여<br>
  이전 방문 위치를 기록한 후<br>
  마지막에 vector 에 모아 거꾸로 출력하였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>
#include<limits.h>

using namespace std;

int n, m;
int start, target;

struct infos
{
	int now;
	int nowCount;
	int nowCost;
};

struct Compare
{
	bool operator()(const infos& a, const infos& b)
	{
		return a.nowCost > b.nowCost;
	}
};

void func(unordered_map<int, vector<pair<int, int>>>& umaps)
{
	// 이전에 방문한 노드와 그 비용
	// 이걸 통해 역추적할 예정
	vector<pair<int, int>> visit(n + 1);

	for (auto& p : visit)
	{
		p.first = 0;
		p.second = INT_MAX;
	}

	visit[start] = { start,0 };

	priority_queue<infos, vector<infos>, Compare> pq;
	pq.push({ start,1,0 });

	int aCost = 0;
	int aCount = 0;

	while (pq.empty() == false)
	{
		int now = pq.top().now;
		int nowCount = pq.top().nowCount;
		int nowCost = pq.top().nowCost;
		pq.pop();

		if (now == target)
		{
			aCost = nowCost;
			aCount = nowCount;
			break;
		}

		if (umaps.find(now) == umaps.end())
			continue;

		for (auto& p : umaps[now])
		{
			int next = p.first;
			int nCost = p.second;

			if (visit[next].second <= nowCost + nCost)
				continue;

			visit[next].first = now;
			visit[next].second = nowCost + nCost;

			pq.push({ p.first,nowCount + 1, nowCost + p.second });
		}
	}

	cout << aCost << '\n';
	cout << aCount << '\n';

	vector<int> v;
	int idx = target;
	for (int i = 0; i < aCount; i++)
	{
		v.push_back(idx);
		idx = visit[idx].first;
	}

	for (auto i = v.rbegin(); i != v.rend(); i++)
		cout << *i << ' ';

}

int main()
{
	cin.tie(nullptr);
	ios::sync_with_stdio(false);

	cin >> n >> m;

	unordered_map<int, vector<pair<int, int>>> umaps;
	umaps.reserve(n);

	for (int i = 0; i < m; i++)
	{
		int s, t, c;
		cin >> s >> t >> c;
		umaps[s].push_back({ t,c });
	}

	cin >> start >> target;

	func(umaps);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/51227fe5-3cd7-4844-9b27-24699f32c7c8)](https://github.com/user-attachments/assets/51227fe5-3cd7-4844-9b27-24699f32c7c8){: .image-popup}<br>

수많은 메모리 초과의 경우<br>
왜 그럴까... 싶었는데<br>

```cpp
if (visit[next].second < nowCost + nCost)
				continue;
```

조건문을 이렇게 작성하여<br>
같은 비용인 경우도 다시 Pq에 넣었기에<br>
발생한 문제...<br>
<= 로 고쳐주니 통과하였다<br>