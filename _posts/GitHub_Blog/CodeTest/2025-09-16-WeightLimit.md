---
title: "백준 Gold 3 중량제한"
date : "2025-09-16 11:00:00 +0900"
last_modified_at: "2025-09-16T11:00:00"
categories:
  - 코딩 테스트
tags:
  - 유니온 파인드
  - Union Find
  - Disjoint Set
  - 다익스트라
  - 그래프
---

## 중량제한 (백준 Gold 3)
<https://www.acmicpc.net/problem/1939><br>

N개의 섬과 M개의 다리가 주어진다<br>
M개의 다리 정보는<br>
start - to - WeightLimit<br>
로서, 시작점과 목적지까지 '버틸 수 있는' 중량 제한이 주어진다<br>

이후, 시작 섬과 목적 섬이 주어질때<br>
옮길 수 있는 최대 중량을 구하는 문제<br>

## 풀이 방법

일단 '일반적'인 길찾기 문제와는 살짝 다르다<br>
'최단거리 비용' 을 구하는 것이 아니라<br>
다소 '돌아가더라도' 최대한의 무게를 유지하는 것<br>

그 외에도 고려한 점들<br>

- 양방향 + 중복 간선이 들어올 수 있기에<br>
  map을 통하여 목적지 중 가장 중량제한이 높은 다리만을 선택<br>

- 문제에서 주어지는 N과 M의 제한이 상당히 높은 편이기에<br>
  빠른 길찾기 알고리즘을 선택<br>
  '시작점'과 '목적점'이 존재하기에<br>
  다소 탐욕스러운 '다익스트라' 고려<br>
  (가장 '무거운' 값이라는 기준이 존재하기에 선택)<br>

- 다음 노드로 넘어갈때, 현재값과 해당 다리 값을 비교하여<br>
  queue에 작은쪽을 넣어줘야 함<br>

- '방문 여부 파악'을 visit 배열과 '분리 집합' 중 어느것을 사용할까 하다가<br>
  최근에 배운 '분리 집합'을 사용<br>
  조건을 검사하고 parent가 같다면 이미 방문한 곳이므로<br>
  pass<br>
  아니라면 Union<br>

- 그렇기에 종료 조건 검사를 살짝 바꿨다<br>
  '큰 값'을 기준으로 힙을 사용하고 있기에<br>
  이미 ret 값이 현재의 nowValue보다 크다면 while 종료<br>

요점은 '무거운 녀석'부터 pq에서 뽑아내어 검사하고<br>
분리 집합을 통해 '이미 방문'한지를 체크<br>
그리고, pq에서 뽑은 값이 '리턴 예정인 값'보다 '작다면' break<br>
이후 반환<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<unordered_map>
#include<limits.h>

using namespace std;

int n, m;

struct nodeInfo
{
	int now;
	int prev;
	int nowValue;
};

struct Compare
{
	bool operator()(const nodeInfo& a, const nodeInfo& b)
	{
		return a.nowValue < b.nowValue;
	}
};

int find(vector<int>& pv,int x)
{
	if (pv[x] == x)
		return x;

	return pv[x] = find(pv, pv[x]);
}

void Union(vector<int>& pv, int a, int b)
{
	a = find(pv, a);
	b = find(pv, b);

	if (a != b)
		pv[a] = b;
}

int getMaxV(unordered_map<int, unordered_map<int, int>>& maps, int start, int end)
{
	vector<int> pv(n + 1);
	for (int i = 0; i <= n; i++)
		pv[i] = i;

	priority_queue<nodeInfo, vector<nodeInfo>, Compare> pq;
	pq.push({ start,0,INT_MAX });

	int ret = 0;

	while (pq.empty() == false)
	{
		int now = pq.top().now;
		int prev = pq.top().prev;
		int nowV = pq.top().nowValue;
		pq.pop();

		// 값 큰 위주로 돌고 있는데 이미 반환 예정값보다 작다면 더 돌필요 없음
		if (ret > nowV)
			break;
		
		if (now == end)
		{
			ret = nowV;
			continue;
		}

		// 이미 방문한 곳인디?
		if (find(pv, now) == find(pv, prev))
			continue;

		Union(pv, now, prev);

		for (auto& p : maps[now])
		{
			int n = p.first;
			int nV = p.second;
			if (nV > nowV)
				nV = nowV;

			pq.push({ n,now,nV });
		}
	}

	return ret;
}

int main()
{
	cin >> n >> m;

	unordered_map<int, unordered_map<int,int>> maps;

	for (int i = 0; i < m; i++)
	{
		int s, t, c;
		cin >> s >> t >> c;

		if (maps[s][t] < c)
			maps[s][t] = c;

		if (maps[t][s] < c)
			maps[t][s] = c;
	}

	int s, t;
	cin >> s >> t;

	cout << getMaxV(maps, s, t);

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/44f642cc-7e69-4c8a-92fb-1add1746bfad)](https://github.com/user-attachments/assets/44f642cc-7e69-4c8a-92fb-1add1746bfad){: .image-popup}<br>

틀린 것은 사소한 오타로 인하여 발생한 것이다<br>

```cpp
if (nowV == end)
{
	ret = nowV;
	continue;
}
```

이렇게 되어 있었는데 예제를 통과하여<br>
그대로 제출한... 그런 실수<br>

분리 집합을 사용하긴 했지만 사실 visit 쪽이<br>
더 가독성이 좋으며, 이러한 '1차적'인 시작->목적지 에는<br>
적합할지도 모른다고 생각한다<br>