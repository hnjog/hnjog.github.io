---
title: "백준 Gold 5 입국심사"
date : "2025-09-20 10:30:00 +0900"
last_modified_at: "2025-09-20T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 매개 변수 탐색
  - 이분 탐색
---

## 입국심사 (백준 Gold 5)
<https://www.acmicpc.net/problem/3079><br>

m명의 친구들이 여행을 가는데<br>
n개의 심사대가 존재한다<br>

n개의 심사대에 걸리는 시간들이 각각 존재할때<br>
m명의 친구들이 최소한의 시간으로 심사대를 통과하는 시간을 구하는 문제<br>

## 풀이 방법

얼핏 보기에는 큐를 이용한 시뮬레이션 문제로 보이나<br>
'제한 조건'을 보면 실로 살벌하다<br>
(n이 10만, m이 10억이 들어올 수 있음)<br>
그리고 T 역시 10^9로 들어올 수 있기에<br>

큐를 사용하여 구현하였더니 자꾸 무엇인가 어긋나는 느낌을 받았다<br>

그렇기에 '매개 변수 탐색'을 이용하여 풀게 되었다<br>

- '정답'을 정해놓고 '정답'이 조건에 맞는지를 확인<br>
  맞다면 더 적은값이 가능한지를 확인하고<br>
  아니라면 정답의 수치를 올려 다시 확인하는 것의 반복<br>
  -> 이분 탐색<br>

- 주어지는 '시간'이 이미 고정되어 있다면<br>
  '몇명'의 사람을 **검사 가능** 한지는 생각보다 쉽게 구할 수 있음<br>
  `'주어지는 시간 / 검사 시간'` 을 각 검사대 마다 구하면<br>
  검사대 마다 검사 가능한 사람이 몇명인지 구할 수 있으며<br>
  이를 합하여 m 이상이면 '가능'<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<limits.h>

using namespace std;

long long n, m;
vector<long long> times;
bool IsAcceptTime(long long targetTime)
{
	long long sum = 0;
	for (int i = 0; i < n; i++)
	{
		sum += (targetTime / times[i]);
		if (sum >= m)
			return true;
	}

	return false;
}

int main()
{
	cin >> n >> m;
	for (int i = 0; i < n; i++)
	{
		int t;
		cin >> t;
		times.push_back(t);
	}

	long long left = 0;
	long long right = LLONG_MAX;

	while (left < right)
	{
		long long mid = (left + right) / 2;

		if (IsAcceptTime(mid))
		{
			right = mid;
		}
		else
		{
			left = mid + 1;
		}
	}

	cout << left;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/e942f253-b31e-4c9f-ad66-241046d7e5c9)](https://github.com/user-attachments/assets/e942f253-b31e-4c9f-ad66-241046d7e5c9){: .image-popup}<br>

처음에는 Priority_queue를 통해 구현하려 했으나<br>
자꾸 일부분이 어긋났다<br>

그렇기에 주어진 '힌트'인 이분탐색과 '매개변수 탐색' 부분을 이용하여<br>
풀게 되었다<br>