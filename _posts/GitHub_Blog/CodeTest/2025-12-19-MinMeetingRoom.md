---
title: "백준 Gold 5 최소 회의실 개수"
date : "2025-12-19 10:30:00 +0900"
last_modified_at: "2025-12-19T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그리디
  - 우선순위 큐
---

## 최소 회의실 개수 (백준 Gold 5)
<https://www.acmicpc.net/problem/28424><br>

n개의 회의와<br>
그 회의의 시작시간과 끝 시간이 주어질때<br>
모든 회의를 진행할 수 있는 최소 회의실 개수를 구하는 문제<br>

- 시작 시간과 끝 시간은 겹칠 수 있음<br>

## 풀이 방법  

주어진 조건을 만족하는 최소한의 자원(방 개수)을 구하는 문제<br>

- **선택 정의**<br>
  : '새로운 방'을 잡을지 or '사용하고 남은 빈 방'을 사용할지<br>

- **선택 기준**<br>
  : 가장 빨리 끝나는 방을 사용할 수 있다면 재사용!<br>
    아니라면 새로운 방 잡기!<br>

**세부 풀이**<br>

- 먼저 '시작 시간'을 기준으로 오름차순 정렬<br>
  - 결국 들어오는 시간 자체는 '고려'해야 함<br>

- 우선순위 큐에 집어넣을 데이터?<br>
  : 해당 회의가 '끝나는 시간'<br>
    - 이미 회의가 진행중이라면 '끝나는 시간'만이 중요해짐<br>
	- 그렇기에 '들어온 회의'들 중 '가장 빨리 끝나는 시간'을 체크해야 함<br>

**정리**<br>

1. 시작 시간을 기준으로 데이터를 오름차순<br>
2. 하나씩 진행하며, pq(끝나는 시간 오름차순)에 집어넣기<br>
3. 집어넣기 전에 '시작시간'과 pq.top()를 비교하여 작다면 pq에서 제거<br>
4. 가장 pq가 커진 타이밍이 '최대'로 빌려야할 '회의실 개수' = 모든 회의 진행 가능한 최소 회의실 개수<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>

using namespace std;

typedef pair<int, int> pii;

int main()
{
	int n;
	cin >> n;

	vector<pii> tv(n);

	for (int i = 0; i < n; i++)
	{
		cin >> tv[i].first >> tv[i].second;
	}

	sort(tv.begin(), tv.end(), []
	(const pii& a, const pii& b)
		{
			if (a.first == b.first)
				return a.second < b.second;

			return a.first < b.first;
		}
	);

	int maxV = 0;
	priority_queue<int,vector<int>,greater<int>> pq;

	int nowtime = 0;

	for (const pii& p : tv)
	{
		if (pq.empty() == false)
		{
			while (pq.empty() == false &&
				pq.top() <= p.first)
			{
				pq.pop();
			}
		}

		pq.push(p.second);

		if (maxV < pq.size())
			maxV = pq.size();
	}

	cout << maxV;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/4aea90c6-f689-4b31-b362-7b59ca802e36)](https://github.com/user-attachments/assets/4aea90c6-f689-4b31-b362-7b59ca802e36){: .image-popup}<br>

문제가 어떤 문제일지 모르는 상황에서<br>
그리디 문제임을 파악하는 것도 중요하다<br>

- 그리디 문제는 보통<br>
  '정렬'을 통하여 순서를 '결정'한 후<br>
  최선의 선택을 통해 푸는 방식<br>

- 또한, 매 선택에 '과거'를 복잡하게 기억할 필요가 없음<br>
  (dp까지는 고려하지 않아도 된다)<br>

- '최소/최대' + '겹침/자원/횟수'를 구하는 문제라면<br>
  그리디 문제일 가능성이 높으므로 유의할것!<br>
