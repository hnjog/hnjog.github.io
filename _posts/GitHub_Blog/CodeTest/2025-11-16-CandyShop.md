---
title: "백준 Gold 4 사탕 가게"
date : "2025-11-16 10:30:00 +0900"
last_modified_at: "2025-11-16T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 사탕 가게 (백준 Gold 4)
<https://www.acmicpc.net/problem/4781><br>

사탕의 종류 N과 돈의 양 M이 주어진다<br>
사탕의 칼로리와 가격이 주어질 때<br>
M으로 만들 수 있는 가장 높은 칼로리를 구하기<br>

## 풀이 방법

기본적인 배낭 문제 타입의 DP 문제<br>
다만, M이 소수점으로 주어지기에 캐스팅을 잘 해야한다<br>

*100 + 0.5를 통해 int 캐스트하여 다른 <br>
0/1 배낭 문제처럼 풀 수 있다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	while (true)
	{
		int n;
		double m;
		cin >> n >> m;

		if (n == 0)
			break;

		vector<int> weights;
		vector<int> vals;

		for (int i = 0; i < n; i++)
		{
			int t;
			double w;
			cin >> t >> w;
			vals.push_back(t);
			weights.push_back(w * 100 + 0.5);
		}

		int mv = m * 100 + 0.5;

		vector<int> dp(mv + 1);

		for (int i = 0; i < n; i++)
		{
			for (int j = weights[i]; j <= mv; j++)
			{
				dp[j] = max(dp[j], dp[j - weights[i]] + vals[i]);
			}
		}

		cout << dp[mv] << '\n';
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/12820209-c27a-411e-b7db-a657b82ff35a)](https://github.com/user-attachments/assets/12820209-c27a-411e-b7db-a657b82ff35a){: .image-popup}<br>

여담으로<br>
*100 + 0.5를 해주는 이유는<br>
'일반적인' 부동 소수점 표기에 따라서<br>
29.0 -> 28.9.... 이런 식으로<br>
실제 내부에 표기될 가능성이 존재하며<br>
해당 부동 소수점 값이 커질 수록, 이러한 사소한 오차가 발생 가능하다<br>
(그렇기에 원본보다 아주 살짝 작은 값이 저장)<br>

- 그렇기에 0.5를 더하여<br>
  아주 살짝 작은 값을 원본보다 크게 만들되<br>
  int 캐스트 될때, 반올림되지는 않게 값을 보정 하는 것<br>
