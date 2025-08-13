---
title: "백준 Gold 2 보석 도둑"
last_modified_at: "2025-08-13T10:00:00"
categories:
  - 코딩 테스트
tags:
  - 그리디 알고리즘
  - 우선순위 큐
---

## 보석 도둑 (백준 Gold 2)
<https://www.acmicpc.net/problem/1202><br>

그리디 문제이다<br>

다만...<br>
개인적으로 정말 고심한 부분들이 많다<br>

불평을 하자면<br>
그리디 알고리즘 문제는 매우 까다로운 것 같다<br>

1. 그리디의 조건을 만족하는가<br>
   (최적 부분 구조와 함께<br>
   '되돌아가서 풀 필요'가 정말로 있는가?라는<br>
   지금 상태로 선택하더라도 최선의 값 아닐까?<br>
   라는 판별이 어렵다)<br>

2. 정렬의 기준 잡기<br>
   	보통 Greedy는 '정렬'을 기반으로<br>
   	1회성으로 선택하여 나가기에<br>
	정렬의 기준을 잘못잡으면<br>
	정답에 닿을듯 말듯한 오답이 탄생한다<br>

3. 자료구조, 알고리즘 선택<br>
   다양한 자료구조에 숙달이 되어 있어야<br>
   문제에서 요구하는 것이 무엇인지 판별 가능하다<br>
   ex)<br>
   현재 시점에서 '최선'의 값 뽑기 : 힙(우선순위 큐)<br>
   빠른 접근 과 카운팅 : Hash 계열 (Unordered_Map 등)<br>
   조건에 맞는 가장 빠른 탐색 : 이진탐색(정렬을 함으로서 선택 가능)<br>

대부분 시간 복잡도가 O(n^2)이 되어버리면<br>
정답과 멀어지기에 무척 어렵다<br>
(보통 이때는 그리디가 아니라 브루트포스에 가까워지긴 하지만)<br>

### 제출 코드 (시간 초과)
```
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>
#include<map>

using namespace std;

typedef unsigned long long ull;
typedef pair<ull, ull> pii;

struct Compare
{
	bool operator()(const pii& a, const pii& b)
	{
		if (a.first != 0 && b.first != 0)
		{
			double av = a.second / a.first;
			double bv = b.second / b.first;

			if (av == bv)
			{
				if (a.second == b.second)
					return a.first > b.first;

				return a.second < b.second;
			}

			return av < bv;
		}
		
		return a.second < b.second;
	}
};

int main()
{
	int n, k;
	cin >> n >> k;

	vector<pii> jewels(n);
	vector<ull> packs(k);

	for (int i = 0; i < n; i++)
	{
		cin >> jewels[i].first; // 무게
		cin >> jewels[i].second; // 가치
	}

	for (int i = 0; i < k; i++)
	{
		cin >> packs[i];
	}

	sort(jewels.begin(), jewels.end(), [](const pii& a, const pii& b)
		{
			if (a.first == b.first)
				return a.second > b.second;

			return a.first > b.first;
		});

	sort(packs.begin(), packs.end());

	ull lastPackSize = packs.back();

	priority_queue<pii, vector<pii>, Compare> pq;
	for (auto& p : jewels)
	{
		if (p.first > lastPackSize)
			continue;

		pq.push(p);
	}

	map<ull,int> valueMap;
	for (auto p : packs)
		valueMap[p]++;

	ull sum = 0;

	while (pq.empty() == false)
	{
		pii p = pq.top();
		pq.pop();
		
		auto it = valueMap.begin();
		bool bFind = false;

		for (; it != valueMap.end(); it++)
		{
			if (it->first >= p.first)
			{
				sum += p.second;
				it->second--;
				break;
			}
		}

		if(it != valueMap.end() && it->second <= 0)
			valueMap.erase(it);

		if (valueMap.empty())
			break;
	}

	cout << sum;

	return 0;
}
```

map을 통하여 가장 적절한 가방을 선택하는 방식<br>
예제에 대하여 정확한 값이 나오지만<br>
내부에서 O(n^2)가 되어버려 시간초과가 발생하였다<br>


### 제출 코드 (오답)
```
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>

using namespace std;

typedef unsigned long long ull;
typedef pair<ull, ull> pii;

struct Compare
{
	bool operator()(const pii& a, const pii& b)
	{
		if (a.first == b.first)
			return a.second < b.second;

		return a.first < b.first;
	}
};

int main()
{
	int n, k;
	cin >> n >> k;

	vector<pii> jewels(n);
	vector<ull> packs(k);

	for (int i = 0; i < n; i++)
	{
		cin >> jewels[i].first; // 무게
		cin >> jewels[i].second; // 가치
	}

	for (int i = 0; i < k; i++)
	{
		cin >> packs[i];
	}

	sort(jewels.begin(), jewels.end(), [](const pii& a, const pii& b)
		{
			if (a.first == b.first)
				return a.second > b.second;

			return a.first > b.first;
		});

	sort(packs.begin(), packs.end(), greater<int>());

	ull lastPackSize = packs[0];

	priority_queue<pii, vector<pii>, Compare> pq;
	for (auto& p : jewels)
	{
		if (p.first > lastPackSize)
			continue;

		pq.push(p);
	}

	ull sum = 0;
	int pIdx = 0;

	while (pq.empty() == false)
	{
		pii p = pq.top();
		pq.pop();
		if (p.first <= packs[pIdx])
		{
			sum += p.second;
			pIdx++;
		}

		if (pIdx >= k)
			break;
	}

	cout << sum;

	return 0;
}
```

보석을 '무게' 위주로 정렬하고<br>
가방을 큰 순서대로 정렬<br>
이후 heap에서 가벼운 순서로 정렬하며<br>
heap에서 나오는 순서대로 값을 구하는 방식<br>

다만, 정렬의 기준이 잘못되었고<br>
'힙'을 제대로 사용하지 못하였다<br>

## 풀이 방식

여러 검색 후<br>
'코드'는 보지 않고<br>
가능한 조언에 집중하여 풀어보았다<br>

### 정렬 순서

- 가방은 '오름차순' 정렬<br>
  : '작은 가방'이 보석을 놓치는 경우를 막아야 하기에 가방은 오름차순 정렬<br>

- 보석은 '무게' 기반의 '오름차순' 정렬<br>
  : 마찬가지로 작은 보석을 먼저 정렬시켜 가방에 넣을 준비를 한다<br>

- 힙은 '가치' 기반 내림차순 정렬<br>
  : 결국 가방에 담아야 하는 것은 '담을 수 있는 가장 비싼 물건'<br>

### 힙에 요소를 넣는 조건

이전 로직을 보니 이 부분이 가장 크게 실수한 부분이었다<br>
'힙'은 '현재 시점'에서 '최선 값'을 구할 수 있는 자료구조인데<br>
그냥 처음부터 모든 보석을 다 넣고 시작한 것이 문제<br>

- 보석용 인덱스 포인터를 하나 더 사용하여(투 포인터)<br>
  '현재 가방의 무게로 담을 수 있는 보석'까지 진행하며<br>
  가치 기반 힙에 넣어준다<br>

- 이후 힙의 가장 위에 올라오는 것은<br>
 '현재 가방이 넣을 수 있는 무게'의<br>
  최대 가치의 보석<br>

- 다만 힙이 비어있다면,<br>
  현재 가방에 넣을 수 있는 보석이 없다는 뜻이므로<br>
  현재의 가방은 무시하게 된다<br>

- 다음 가방으로 넘어가고 새로운 보석이 힙에 들어와도<br>
  '가방에 넣을 수 있는 무게' 중 '가장 가치 있는 보석'이<br>
  top에 위치하기에 가방마다 최적의 보석을 고를 수 있게 된다<br>

## 결과

<img width="1155" height="406" alt="Image" src="https://github.com/user-attachments/assets/a8112aa5-0701-4471-83fb-a7669f90f190" /><br>

그리디 문제는 항상 쉽게 풀 수 있는 것 처럼 보이지만<br>
절대 아님을 통감하였다<br>

### 제출 코드

```
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>

using namespace std;

typedef unsigned long long ull;
typedef pair<ull, ull> pii;

struct Compare
{
	bool operator()(const pii& a, const pii& b)
	{
		return a.second < b.second;
	}
};

int main()
{
	int n, k;
	cin >> n >> k;

	vector<pii> jewels(n);
	vector<ull> packs(k);

	for (int i = 0; i < n; i++)
	{
		cin >> jewels[i].first; // 무게
		cin >> jewels[i].second; // 가치
	}

	for (int i = 0; i < k; i++)
	{
		cin >> packs[i];
	}

	sort(jewels.begin(), jewels.end(), [](const pii& a, const pii& b)
		{
			return a.first < b.first;
		});

	sort(packs.begin(), packs.end());

	priority_queue<pii, vector<pii>, Compare> pq;

	ull sum = 0;
	int jIdx = 0;

	for (int i = 0; i < k; i++)
	{
		// 현재 가방 무게보다 작은 jewel들을 가치 기반 heap에 담는다
		while (jIdx < n &&
			jewels[jIdx].first <= packs[i])
		{
			pq.push(jewels[jIdx]);
			jIdx++;
		}

		if (pq.empty() == false)
		{
			sum += pq.top().second;
			pq.pop();
		}
	}

	cout << sum;

	return 0;
}
```