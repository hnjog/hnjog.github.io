---
title: "백준 Gold 3 일감호에 다리 놓기"
date : "2026-02-25 12:00:00 +0900"
last_modified_at: "2026-02-25T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 최소 스패닝 트리
---

## 일감호에 다리 놓기 (백준 Gold 3)
<https://www.acmicpc.net/problem/17490><br>

모든 강의동을 순회할 수 있는지를 체크하는 문제<br>

*유의할 점*<br>
- 특정한 공사 구간이 존재함<br>
  - 공사 구간은 n, n+1 처럼 서로 연결된 구역<br>
  - 이 구역으로는 다닐 수 없음<br>
- 중앙 호수에 작은 섬이 존재하여 이 섬에 징검다리를 놓을 수 있음<br>
  - 각 섬들에 필요한 '돌 개수' 존재<br>
- 1동과 N동은 기본적으론 연결되어 있음 (원형)<br>

이러한 조건에서 '돌의 개수'가 주어졌을 때<br>
모든 강의동을 순회할 수 있다면 "YES", 그렇지 않다면 "NO"를 출력하는 문제<br>

## 풀이 방법  

'특정 조건' 하에, 모든 노드가 연결되었는지를 묻는 MST 계열의 문제라 생각하였다<br>

*유의할 점*<br>
- 이 문제는 '돌의 개수'가 넉넉한지를 묻는게 아니라, 순회 여부를 묻는것!<br>
  - 따라서 '공사 구간'의 개수가 1이하라면 사실상 YES 이다<br>
  - 0개라면 굳이 돌을 안놓아도 되며, 1개라면 다른 방향으로 돌아갈 수 있기에<br>
    (원형!)<br>

*풀이 순서*<br>
1. 초기 비용을 기반으로 Edge 생성하기<br>
2. Pq에 넣기(비용 오름차순)<br>
3. 단절되지 않은 이웃된 공간을 비용 0으로 pq에 넣기<br>
4. 연결 시도하기<br>
5. 최종 코스트를 기반으로 출력<br>

*고민한 점*<br>
- 개인적으로 3번의 '단절'을 파악하는 방식이 제일 까다로웠다<br>
- '단절'된 정보만이 주어지지만, 실제 필요한 것은 '연결'된 것...<br>
  - 그렇기에 unordred_map을 통해 초기의 데이터를 보관한 후<br>
    '단절'된 것을 제거하는 방식으로 처리를 해보았다<br>
  - 초기에는 *set*를 고려했지만, 결국 시간초과가 발생하여<br>
    unordered_map으로 수정하여 풀 수 있었다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>
#include<unordered_map>

using namespace std;

typedef pair<int, int> pii;

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
	long s, t, cost;
};

struct Compare
{
	bool operator()(const edge& a, const edge& b)
	{
		return a.cost > b.cost;
	}
};

int main()
{
	long n, m, k;
	cin >> n >> m >> k;

	vector<int> pv(n + 1, 0);
	for (int i = 0; i <= n; i++)
	{
		pv[i] = i;
	}

	priority_queue<edge, vector<edge>, Compare> pq;
	unordered_map<long, long> um;

	for (int i = 1; i <= n; i++)
	{
		long t;
		cin >> t;
		
		pq.push({0,i,t});
		if (i != n)
			um[i] = i + 1;
		else
			um[i] = 1;
	}

	for (int i = 0; i < m; i++)
	{
		long a, b;
		cin >> a >> b;

		if (um.find(a) != um.end() &&
			um[a] == b)
		{
			um.erase(a);
		}
	}

	for (auto& p : um)
	{
		pq.push({ p.first,p.second,0 });
	}

	long s = 0;

	// pq 돌면서 체크하기
	while (pq.empty() == false)
	{
		const edge& e = pq.top();
		if (Union(pv, e.s, e.t))
		{
			s += e.cost;
		}
		pq.pop();
	}

	// 공사중이 하나라 이하라면 그냥 돌아갈 수 있음
	if (m <= 1)
		s = 0;

	if (s <= k)
	{
		cout << "YES";
	}
	else
	{
		cout << "NO";
	}

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/4c876de5-b6cb-4d25-aabe-3454eb72c761)](https://github.com/user-attachments/assets/4c876de5-b6cb-4d25-aabe-3454eb72c761){: .image-popup}<br>

MST 계열 문제는 '분리 집합'의 응용만 생각했었지만<br>
애초에 Edge에 대한 처리를 생각할 수 있었던 좋은 문제라 생각한다<br>
