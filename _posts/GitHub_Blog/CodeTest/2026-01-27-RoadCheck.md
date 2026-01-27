---
title: "백준 Gold 1 도로검문"
date : "2026-01-27 12:00:00 +0900"
last_modified_at: "2026-01-27T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 다익스트라
  - 최단 경로
  - 그래프
  - 역추적
---

## 도로검문 (백준 Gold 1)
<https://www.acmicpc.net/problem/2307><br>

n개의 노드와 m개의 간선 정보가 주어지고<br>
도둑과 경찰이 존재한다<br>

- 도로(공통)<br>
  - a,b,c 로 나뉘며 각각 시작,끝,시간 으로 구분<br>
  - 양방향 도로이며, '유일'하다<br>

- 도둑<br>
  - 도시의 1번에 도착하여 N번으로 나감<br>
  - 이 과정에서 '최단거리'를 소요하는 경로를 통해 나가게 됨<br>

- 경찰<br>
  - 간선을 '한 개' 막을 수 있음<br>
  - 막힌 간선은 도둑이 경로로 선택하지 못함<br>
  - 그렇기에 '원래 도둑의 최단거리 탈출 시간'을 최대한 지연시키는 것이 목적<br>

- 문제가 원하는 것<br>
  - 탈출 지연 시간 : '경찰이 임의의 도로를 막았을때의 도둑 탈출 시간' - '원래 도둑 탈출 시간'<br>
  - '가장 큰' 탈출 지연 시간을 구함<br>
  - 다만 '임의의 도로'를 막았을때, 도둑이 탈출 불가능하다면 -1을 출력!<br>

## 풀이 방법  

다익스트라 변형 문제!<br>

처음에는 단순히 '다익스트라'에<br>
'특정 간선'을 선택하지 않음으로서 풀 수 있는 문제라고 생각하였으나<br>
다음 질문으로 곧 생각을 바꾸게 되었다<br>

- 특정 '간선' 을 선택하지 않았을 때의 '지연 시간'을 어떻게 구하지..??<br>

고민하다 '힌트'를 보았는데<br>
'역추적'이라는 키워드를 보고 생각에 잠기었다<br>

일정시간 고민을 한 후, 다음과 같은 풀이 방식을 세우게 되었다<br>

- 다익스트라를 '먼저' 한번 돌리고<br>
  도둑들이 '최단 거리'를 통과하기 위한 '최단 경로'를 구함<br>
  - '역추적'을 통해 구할 수 있음<br>

- 이후 그 '최단 거리' 들에 대한 edge들을 선택하고<br>
  하나씩 막으면서 '다익스트라'를 돌리면서<br>
  '가장 높은 비용'을 구하기<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>

using namespace std;

typedef pair<int, int> pii;

struct infos
{
	int now;
	int nowc;
};

struct Compare
{
	bool operator()(const infos& a, const infos& b)
	{
		return a.nowc > b.nowc;
	}
};

vector<pii> getvv(const int n, unordered_map<int, unordered_map<int, int>>& maps)
{
	priority_queue<infos, vector<infos>, Compare> pq;
	pq.push({ 1,0 });

	vector<pii> ret(n + 1, {1e9,1e9});
	ret[1].first = 1;
	ret[1].second = 0;

	while (pq.empty() == false)
	{
		int now = pq.top().now;
		int nowc = pq.top().nowc;
		pq.pop();

		if (now == n)
		{
			break;
		}

		for (auto& p : maps[now])
		{
			int ne = p.first;
			int nec = p.second;

			if (ret[ne].second <= nowc + nec)
				continue;

			ret[ne].first = now;
			ret[ne].second = nowc + nec;
			pq.push({ ne,nowc + nec });
		}
	}

	return ret;
}

int GetV(const int n, unordered_map<int, unordered_map<int, int>>& maps, pii banned)
{
	priority_queue<infos, vector<infos>, Compare> pq;
	
	pq.push({ 1,0 });

	vector<int> visited(n + 1, 1e9);

	while (pq.empty() == false)
	{
		int now = pq.top().now;
		int nowc = pq.top().nowc;
		pq.pop();

		if (visited[now] <= nowc)
			continue;

		visited[now] = nowc;

		if (now == n)
		{
			return nowc;
		}

		for (auto& p : maps[now])
		{
			int ne = p.first;
			int nec = p.second;

			if ((banned.first == ne && banned.second == now) ||
				(banned.first == now && banned.second == ne))
				continue;

			pq.push({ ne,nowc + nec });
		}
	}

	return -1;
}

int GetFV(const int n, const int m, unordered_map<int, unordered_map<int, int>>& maps)
{
	// 1. 일반 pq로 경로 와 최단거리 구하기
	vector<pii> load = getvv(n, maps);
	int mv = load[n].second;

	// 2. 금지 경로 저장
	vector<pii> banLoad;
	banLoad.reserve(m);

	int idx = n;
	while (load[idx].first != idx)
	{
		banLoad.push_back({ idx,load[idx].first });
		idx = load[idx].first;
	}

	// 3. 설정된 금지 경로들을 바탕으로 다시 시작점(1)에서부터
	// pq 돌리기

	int ret = 0;

	for (pii& p : banLoad)
	{
		int t = GetV(n, maps, p);
		if (t == -1)
			return -1;

		ret = max(ret, t - mv);
	}

	return ret;
}

int main()
{
	int n, m;
	cin >> n >> m;

	unordered_map<int, unordered_map<int, int>> maps;
	maps.reserve(n);

	for (int i = 0; i < m; i++)
	{
		int a, b, c;
		cin >> a >> b >> c;
		maps[a][b] = c;
		maps[b][a] = c;
	}

	vector<pii> outs;
	outs.reserve(n + 1);

	cout <<  GetFV(n,m, maps);

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/5dfcfb8e-a113-4925-8e73-995313ac38c8)](https://github.com/user-attachments/assets/5dfcfb8e-a113-4925-8e73-995313ac38c8){: .image-popup}<br>

- 이제 보니 굳이 `unordered_map` 을 value로 쓸 필요가 없었을지도?<br>
  그냥 `vector<pair<int,int>>`여도 충분히 풀 수 있었을 것 같다<br>

- edge를 한 번에 돌릴 수 없나 고민하였지만<br>
  결국 '-1' 출력 조건에 고심하여 반복문을 도는 쪽으로 풀게 되었다<br>
