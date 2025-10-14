---
title: "백준 Gold 5 행복 유치원"
date : "2025-10-15 10:30:00 +0900"
last_modified_at: "2025-10-15T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그리디
  - 우선순위 큐
---

## 행복 유치원 (백준 Gold 5)
<https://www.acmicpc.net/problem/13164><br>

주어지는 n개의 수들을 k개로 나누면<br>
각 조의 가장 작은수와 큰 수의 차이를 가지는 비용이 발생한다<br>
이 비용이 가장 적게 발생했을때의 비용을 구하는 문제<br>

## 풀이 방법

- 알아둘 것은<br>
  오름차순으로 정렬된 특정 조 a 에서<br>
  a1... a2... a3 가 존재할때<br>
  `a3 - a1 = a3 - a2 + a2 - a1` 과 같다는 점이다<br>
  (즉, **인접한 수끼리 연산을 진행하여도 결과적으론 문제 없다**)<br>
  (-> 그리디 하다)<br>

- 그렇기에 주어진 수들을 정렬하고<br>
  그 각각의 '차'를 미리 저장한다<br>
  (정답에 더해주며, 최대힙에 넣어준다)<br>

- 조를 나누는 수가 k 이므로<br>
  k-1 개의 '차'를 제거할 수 있음<br>
  (k가 1인 경우는 모두 한조여야 하므로)<br>

- 최대힙에서 '가장 큰 차' 들을 k-1개만큼 꺼내<br>
  정답에서 빼주면 문제 해결!<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<algorithm>

using namespace std;

int main()
{
	int n, k;
	cin >> n >> k;

	int ans = 0;

	vector<int> vecs(n);
	priority_queue<int> pq;

	for (int i = 0; i < n; i++)
	{
		cin >> vecs[i];
	}

	sort(vecs.begin(), vecs.end());

	for (int i = 1; i < n; i++)
	{
		int v = vecs[i] - vecs[i-1];
		ans += v;
		pq.push(v);
	}

	int count = k - 1;

	while (count > 0)
	{
		ans -= pq.top();
		pq.pop();
		count--;
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/fcd46474-f3da-45aa-83f8-57ff7b5570de)](https://github.com/user-attachments/assets/fcd46474-f3da-45aa-83f8-57ff7b5570de){: .image-popup}<br>

사실 어제 푼 문제에서 힌트를 얻은 점이 있다<br>

- 미리 결과물을 더한 후<br>
  모든 결과 중 가장 불필요한 것을 제거<br>

해당 방식을 통해<br>
각 수들의 '차'를 구한 후, 전부 더해놓고<br>
그 '차'들을 최대힙에 집어넣어<br>
꺼내고 빼줌으로서 문제를 풀 수 있었다<br>

