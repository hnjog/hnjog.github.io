---
title: "백준 Gold 4 수도배관공사"
date : "2025-11-26 10:30:00 +0900"
last_modified_at: "2025-11-26T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 수도배관공사 (백준 Gold 4)
<https://www.acmicpc.net/problem/2073><br>

필요한 수도관 길이 d와 파이프 개수 p가 주어진다<br>

파이프의 길이, 용량이 주어질 때<br>
d를 통해 만들 수 있는 파이프 용량 중 가장 큰 것을 구하는 문제<br>

- 파이프 길이의 합이 딱 d 길이여야 함<br>
	- 최소 1개는 d 길이를 만들 수 있도록 데이터가 주어짐<br>

- 연결된 파이프는 '작은 쪽'의 용량에 맞춰짐<br>

## 풀이 방법

dp문제이기에 각 진행 방식을 검토하여 풀어보았다<br>

**점화식**<br>
- dp[i] : i값으로 만들 수 있는 파이프 용량의 최댓값<br>

**다음 상태 이동**<br>
- dp의 초기 값 세팅은 -1로 하되<br>
  dp[0]의 경우만 무한대 값으로 세팅<br>
  - dp[n] 을 -1로 세팅하여 '만들 수 없음'을 암시<br>
  - 반대로 dp[0]은 0값 자체의 특이성을 세팅하는 용도<br>

- min과 max 를 적용할 대상을 구분하기<br>
  - min : dp[j - len[i]]를 현재 cap[i] 과 비교하여 작은 값 선택 (파이프는 작은 쪽의 용량을 따르므로)<br>
  - max : dp[j]와 minValue 를 비교하여 큰 값 선택 (dp[j] 중 큰 값을 구하는 것이 문제의 목표이므로)<br>

- dp[n]이 -1 이므로 min 부분에서 만들수 없는 경우가 걸러짐<br>
- dp[0]이 무한대 이므로, min 부분에서 만들 수 있는 수치를 시작할 수 있음<br>

**순회 방식**<br>
- dp가 1차원이며, 개수 제한이 있는 dp이므로<br>
  내림차순을 통해 각 요소마다 dp[n]에 한번씩만 접근 가능하도록 제한<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<limits.h>

using namespace std;

int main()
{
	int d, p;
	cin >> d >> p;

	vector<long> lens, caps;

	for (int i = 0; i < p; i++)
	{
		long l, c;
		cin >> l >> c;
		lens.push_back(l);
		caps.push_back(c);
	}

	vector<long> dp(d + 1,-1);
	dp[0] = INT_MAX;

	for (int i = 0; i < p; i++)
	{
		long nowl = lens[i];
		long nowc = caps[i];

		for (int j = d; j >= nowl; j--)
		{
			long mv = min(dp[j - nowl], nowc);

			dp[j] = max(dp[j], mv);
		}
	}

	cout << dp[d];

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/43b2e95d-899d-4eba-995e-1d7b5dc64f2f)](https://github.com/user-attachments/assets/43b2e95d-899d-4eba-995e-1d7b5dc64f2f){: .image-popup}<br>

처음에는 2차원 dp[i][j]를 통해 풀어 메모리 초과가 발생하였다<br>

그러나 이후,<br>
1차원 dp로도 충분히 가능할 것 같다는 생각이 들었기에<br>
해당 방식을 적용하여 풀어보았다<br>

- 푸는 방식 자체는 일반적인 배낭 문제였으나<br>
  상태 전이 부분을 잘 응용해야 풀 수 있는 문제였다<br>
