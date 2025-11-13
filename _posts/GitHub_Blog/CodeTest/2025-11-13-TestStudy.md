---
title: "백준 Gold 5 벼락치기"
date : "2025-11-13 10:30:00 +0900"
last_modified_at: "2025-11-13T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 벼락치기 (백준 Gold 5)
<https://www.acmicpc.net/problem/14728><br>

문제의 개수 N과 총 푸는 시간 T가 주어진다<br>
문제마다 시간 K와 배점 S가 주어질때<br>
T를 최대한 활용한 최대한의 점수를 구하는 문제<br>

## 풀이 방법
`다이나믹 프로그래밍`, 그 중 '배낭 문제'이다<br>

- 주어지는 시간 T가 배낭의 총량<br>

- 각 문제의 시간 K가 '무게',<br>
  그 배점 S 가 '가치'로 해석하면 배낭 문제로 적용하여 풀 수 있음<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n, t;
	cin >> n >> t;
	
	vector<int> weight, value;

	for (int i = 0; i < n; i++)
	{
		int w, v;
		cin >> w >> v;
		weight.push_back(w);
		value.push_back(v);
	}
	
	vector<int> dp(t + 1,0);

	for (int i = 0; i < n; i++)
	{
		for (int j = t; j >= weight[i]; j--)
		{
			dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
		}
	}

	cout << dp[t];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/8394d5b1-724c-4aad-8e08-55b17f6d65ea)](https://github.com/user-attachments/assets/8394d5b1-724c-4aad-8e08-55b17f6d65ea){: .image-popup}<br>

```cpp
for (int i = 0; i < n; i++)
{
	for (int j = t; j >= weight[i]; j--)
	{
		dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
	}
}
```

배낭 문제에서 단순히 <br>
수치만을 구할때 유용한 코드 스니펫이다<br>

- 최대 수치 T에서 천천히 내려오며, 해당 회차의<br>
  무게값과 가치값을 비교하여 적용하며<br>
  dp가 1차원이기에 직관적이다<br>

- j를 역순으로 돌리는 이유는 dp[j]를 한번만 갱신하기 위함<br>

- 2차원 dp 방식의 경우,<br>
  선택의 과정이 이전 dp 들에 남기에<br>
  역추적이 가능해짐<br>
  - ex : dp[i][j]가 dp[i-1][j]와 같다면 현재 순서의 아이템은 선택하지 않은 것<br>

- 1차원 방식도 추가로 vector 를 이용한 방식으로 응용은 가능<br>
