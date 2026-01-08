---
title: "백준 Gold 2 회의준비"
date : "2026-01-08 12:00:00 +0900"
last_modified_at: "2026-01-08T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 그래프 이론
  - 최단 경로
  - 분리 집합
  - BFS
---

## 회의준비 (백준 Gold 2)
<https://www.acmicpc.net/problem/2610><br>

위원회를 구성하는 조건은 다음과 같다<br>

- 서로 알고 있는 사람끼리 위원회에 속해야 함<br>
- 위원회의 수는 최대가 되어야 함<br>

이러한 위원회에서<br>
의견 전달을 위해 '대표'를 뽑아야 한다<br>

- 의견 전달은 '서로 아는 사람'끼리만 가능하기에<br>
  대표는 이러한 '의견 전달' 횟수가 최소여야 한다<br>
  - 더 정확히는, 가장 '멀리 떨어진' 위원과의 거리가 최소여야 함<br>

- 이러한 조건이 같다면 그 중, 아무나 선택할 수 있다<br>

인원수와 관계가 주어질때<br>
위원회의 개수와 대표의 번호를 출력하는 문제<br>

- 대표의 번호는 오름차순으로 출력한다<br>

## 풀이 방법  

처음 생각난 것은 MST였으나<br>
이 문제는 '최소 스패닝 트리'를 구하는 것이 아니고<br>
그 부모를 모아서 구성하는 방식이고<br>
부모를 구하는 과정이 MST와 조금 다른듯 하기에<br>
'분리 집합'만을 사용하기로 하였다<br>

- 분리 집합을 통해<br>
  '특정 조건'을 만족하는 경우, 다른 부모를 섬기도록 하며<br>
  이후, set 을 사용하여 대표의 개수와 오름차순 정렬을 하면 될 것 같았다<br>

- '특정 조건'?<br>
  : '의견 전달'의 최댓값이 최소가 되는 경우<br>
    -> Tree의 최대 Depth가 가장 낮은 경우로 인식하였다<br>

- 그렇기에 각각의 N에 BFS를 진행하여<br>
  해당 인원의 '특정 조건' 값을 구하는 방식으로 진행<br>
  (지금 생각해보면 플로이드-워셜로도 풀 수 있었다고 생각한다)<br>

- 이후, Union을 거치고 set으로 순회하며 부모를 insert 한 후<br>
  개수와 대표를 출력하면 끝!<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<set>
#include<queue>
#include<unordered_map>

using namespace std;

typedef pair<int, int> pii;

int FindP(vector<pii>& pv, int x)
{
	if (pv[x].first == x)
		return x;

	return pv[x].first = FindP(pv, pv[x].first);
}

bool Union(vector<pii>& pv, int a, int b)
{
	a = FindP(pv, a);
	b = FindP(pv, b);

	if (a == b)
		return false;

	if (pv[a].second > pv[b].second)
		pv[a].first = b;
	else
		pv[b].first = a;

	return true;
}

int getV(unordered_map<int, vector<int>>& maps, int start)
{
	int ret = 0;

	queue<pii> nq;
	nq.push({ start,0 });
	set<int> visit;

	while (nq.empty() == false)
	{
		int now = nq.front().first;
		int nowC = nq.front().second;
		nq.pop();

		if (visit.find(now) != visit.end())
			continue;

		visit.insert(now);

		if (nowC > ret)
			ret = nowC;

		for (int n : maps[now])
		{
			nq.push({ n,nowC + 1 });
		}
	}

	return ret;
}

int main()
{
	int n, m;
	cin >> n >> m;

	vector<pii> edges;
	unordered_map<int, vector<int>> maps;

	for (int i = 0; i < m; i++)
	{
		int a, b;
		cin >> a >> b;
		a--;
		b--;
		edges.push_back({ a,b });
		maps[a].push_back(b);
		maps[b].push_back(a);
	}

	vector<pii> pv(n);

	for (int i = 0; i < n; i++)
	{
		pv[i].first = i;
		pv[i].second = getV(maps, i);
	}

	set<int> ret;

	for (pii& p : edges)
	{
		Union(pv, p.first, p.second);
	}

	for (int i = 0; i < n; i++)
	{
		ret.insert(FindP(pv, pv[i].first) + 1);
	}

	cout << ret.size() << '\n';

	for (int r : ret)
	{
		cout << r << '\n';
	}

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/51487d5b-20a0-45dd-93be-f1ace09eb2ad)](https://github.com/user-attachments/assets/51487d5b-20a0-45dd-93be-f1ace09eb2ad){: .image-popup}<br>

문제를 2번정도 잘못 해석한 점이 있었다<br>

1. 분리 집합을 통해, '등장한 개수'만 세어주고 그걸로 묶으면 되겠는데?<br>
2. 아하 거리 계산이 필요하구나? 총 거리를 세어주고 그걸로 비교해야지<br>

그런데 사실 필요한 것은<br>
'최댓값'이 최소가 되게하는 것이 문제가 요구한 조건이었기에...<br>
마지막에 거리 계산 내부를 수정하여 풀 수 있었다<br>

