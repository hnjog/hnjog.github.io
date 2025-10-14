---
title: "백준 Gold 3 순회강연 "
date : "2025-10-14 10:30:00 +0900"
last_modified_at: "2025-10-14T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 그리디
  - 우선순위 큐
---

## 순회강연  (백준 Gold 3)
<https://www.acmicpc.net/problem/2109><br>

교수가 제한된 일정내에 강연을 다니며<br>
가장 많이 돈을 벌 수 있는 수치를 구하는 문제<br>

- 대학에서 진행하는 강의는 n일차 내로 와서 강의를 진행하면<br>
  p 값을 준다<br>

- 강의는 하루가 꼬박 걸린다<br>
  (이동 시간은 고려 x)<br>

## 풀이 방법

문제의 힌트는 '강의'가 하루 걸린다는 점<br>
그리고 pq를 잘 이용한다면<br>
다음과 같은 방식을 고려할 수 있다<br>

- pq의 크기 : 진행한 총 일자<br>

따라서 문제의 접근 방식은<br>

- 일단 강의를 pq에 집어넣으며<br>
  그 p값을 정답에 더함<br>

- 다만 pq의 크기가 현재 강의의 '일자'를 넘긴 경우<br>
  힙에서 가장 작은 값의 크기를 꺼내<br>
  정답에서 빼줌<br>
  (그렇기에 `최소힙`을 사용)<br>

- pq의 사이즈가 곧 '강의'를 실행하는 날자의 총합이므로<br>
  강의 일자가 부족하다면 그 중에서 '하나'를 제거함으로서<br>
  일정을 맞출 수 있음<br>
  (강의가 하루 걸리며, 현재 강의의 일자가 그것을 넘어선다면<br>
  그 중 가장 작은 값을 주는 강의를 안 한것으로 치면 되므로)<br>

## 제출 코드

```cpp
#include<iostream>
#include<queue>
#include<vector>
#include<algorithm>

using namespace std;

struct linfo
{
	int limitDay;
	int pay;
};

struct Compare
{
	bool operator()(const linfo& a, const linfo& b)
	{
		return a.pay > b.pay;
	}
};

int main()
{
	int n;
	cin >> n;

	if (n == 0)
	{
		cout << 0;
		return 0;
	}

	vector<linfo> vec;

	for (int i = 0; i < n; i++)
	{
		linfo l;
		cin >> l.pay >> l.limitDay;
		vec.push_back(l);
	}

	sort(vec.begin(), vec.end(), [](const linfo& a, const linfo& b) 
		{
			if (a.limitDay == b.limitDay)
				return a.pay > b.pay;

			return a.limitDay < b.limitDay;
		});

	int ans = 0;

	priority_queue<linfo, vector<linfo>, Compare> pq;

	for (int i = 0; i < n; i++)
	{
		int ld = vec[i].limitDay;
		int p = vec[i].pay;
		pq.push(vec[i]);
		ans += p;

		if (ld < pq.size())
		{
			ans -= pq.top().pay;
			pq.pop();
		}
	}

	cout << ans;
	
	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/76233c1c-d2a4-4c6a-926d-83abbf34f32c)](https://github.com/user-attachments/assets/76233c1c-d2a4-4c6a-926d-83abbf34f32c){: .image-popup}<br>

처음에는 누적 일수를 기반으로<br>
최대힙에 값을 '담는 강의'를 제한하는 방식으로 진행하였으나<br>
정확도가 떨어져 오답이 발생하였다<br>

이후 문제에 관한 조언을 들은 후<br>
잘못 생각한 부분을 찾을 수 있었고<br>
'최소힙'으로 변경하여 문제를 풀 수 있었다<br>
