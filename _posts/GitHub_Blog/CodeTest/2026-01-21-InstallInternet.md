---
title: "백준 Gold 1 인터넷 설치"
date : "2026-01-21 12:00:00 +0900"
last_modified_at: "2026-01-21T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 다익스트라
  - 최단 경로
  - 그래프
---

## 인터넷 설치 (백준 Gold 1)
<https://www.acmicpc.net/problem/1800><br>

노드의 개수 N, 간선의 개수 P, 무료 비용 K개가 제공될 때<br>
1과 N을 인터넷 연결하는 문제<br>

- 1번은 인터넷이 연결되어 있음<br>
- 간선의 정보 : 시작,도착,비용<br>
- 연결된 모든 간선 비용 중, 가장 비싼 비용만 지불<br>
  - 그런데 K개를 무료로 해주므로, K개 이후 가장 비싼 비용을 지불<br>
- 1번에서 N까지 도달할 수 없다면 -1 출력<br>

## 풀이 방법  

다익스트라 변형 문제!<br>

일반적인 다익스트라와 다른점은 크게 2가지이다<br>

1. k를 사용하여 비용을 없앨수 있음!<br>
2. '가장 비싼 비용'만 지불<br>

그렇기에 관련한 노드 진행에 대한 로직을 일부 수정함으로서<br>
문제를 풀 수 있다!<br>

- k를 사용하여 비용 제거<br>
  - k를 사용하냐, 사용하지 않냐로 문제를 구분해야 함<br>
    (그렇기에 visit를 2차원 배열로 관리)<br>
  - PQ 정렬 조건<br>
    - K를 사용하지 않을수록 우선순위 높아짐<br>
	- 둘다 같다면 '비용'이 적은 순서로 우선순위를 높이도록!<br>
	  -> 결론적으로 'K'를 다 사용한 녀석 중, 비용이 가장 적은 녀석이 '가장 먼저' pq의 위로 올라옴<br>
	     (그리고 이것이 목적지라면 정답!)<br>

- '가장 비싼 비용'만 지불!<br>
  - K를 사용한다면, 현재 비용 + k개수 1감소<br>
    (k를 사용할 수 있다면! '> 0')<br>
  - K를 사용 안한다면, 현재 비용과 다음 비용 중 **큰 값**을 담음<br>

## 제출 코드

```cpp
#include<iostream>
#include<queue>
#include<vector>
#include<unordered_map>

using namespace std;

typedef pair<int, int> pii;

struct infos
{
	int now, nowC;
	int remainK;
};

struct Compare
{
	bool operator()(const infos& a, const infos& b)
	{
		if (a.remainK == b.remainK)
			return a.nowC > b.nowC;

		return a.remainK < b.remainK;
	}
};

int GetMaxV(const int n, const int k, unordered_map<int, vector<pii>>& maps)
{
	priority_queue<infos, vector<infos>, Compare> pq;
	pq.push({ 1,0,k });

	vector<vector<int>> visited(n + 1, vector<int>(k + 1, 1e9));

	while (pq.empty() == false)
	{
		int now = pq.top().now;
		int nowC = pq.top().nowC;
		int nowRK = pq.top().remainK;
		pq.pop();

		if (visited[now][nowRK] <= nowC)
			continue;

		visited[now][nowRK] = nowC;

		if (now == n &&
			nowRK == 0)
		{
			return nowC;
		}

		for (pii& p : maps[now])
		{
			if (nowRK > 0)
				pq.push({ p.first,nowC,nowRK - 1 });

			pq.push({ p.first, max(nowC , p.second),nowRK });
		}
	}

	return -1;
}

int main()
{
	int n, p, k;
	cin >> n >> p >> k;

	unordered_map<int, vector<pii>> maps;

	for (int i = 0; i < p; i++)
	{
		int a, b, c;
		cin >> a >> b >> c;
		maps[a].push_back({ b,c });
		maps[b].push_back({ a,c });
	}

	cout << GetMaxV(n, k, maps);

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/4b6ad4bc-0274-45b0-a390-8d9e406c2cee)](https://github.com/user-attachments/assets/4b6ad4bc-0274-45b0-a390-8d9e406c2cee){: .image-popup}<br>

요새는 문제를 필터링하지 않고<br>
백준의 '힌트'를 보지 않고 풀어보려고 하고 있다<br>
(골드 기준)<br>

- 그런데 그래프 문제가 엄청 많이 보이는..???<br>

