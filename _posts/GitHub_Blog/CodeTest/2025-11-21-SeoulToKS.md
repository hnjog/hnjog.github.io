---
title: "백준 Gold 4 서울에서 경산까지"
date : "2025-11-21 10:30:00 +0900"
last_modified_at: "2025-11-21T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 서울에서 경산까지 (백준 Gold 4)
<https://www.acmicpc.net/problem/14863><br>

이동을 위한 도시 N개, 제한 시간 K가 주어진다<br>
N개에 대하여<br>

- 걷기 소비 시간,획득 가치  <br>
- 자전거 소비 시간,획득 가치  <br>

가 존재할때<br>

K 시간 내에, 목적지로 가며 얻을 수 있는<br>
최대 가치를 구하는 문제<br>

- 반드시 둘 중 하나를 선택하되<br>
  K시간 내로 도달할 수 있는 거리는 주어짐<br>


## 풀이 방법

**점화식**<br>
- dp[i][j] : i번째 선택시까지, 시간 j로 얻을 수 있는 최대 가치<br>
  - 다만 도달 불가능한지의 여부 파악을 위해 -1 로 초기화<br>

**다음 상태 이동**<br>
- dp[i-1][...]에서 초기화 값이라면 다음 상태로 이동하지 않음<br>
- 걷기와 자전거 타기 중 하나를 반드시 선택해야 하므로<br>
  양 수치를 비교하여 체크<br>
  - best = max(best, dp[i - 1][j - walkCost] + walkValue)<br>
  - best = max(best, dp[i - 1][j - cycleCost] + cycleValue)<br>

**순회 방식**<br>
- 2차원 방식이며, 시간 j를 기준으로 오름차순 진행<br>
  - ex) 시간을 진행함으로서 걷는 시간 만족시, 걷기 비용을 획득<br>

이후<br>
마지막으로 순회하면서<br>
가장 높은 값을 찾기<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n, k;
	cin >> n >> k;
	vector<pair<long, long>> walk, cycle;

	for (int i = 0; i < n; i++)
	{
		long w1, w2, c1, c2;
		cin >> w1 >> w2 >> c1 >> c2;
		walk.push_back({ w1,w2 });
		cycle.push_back({ c1,c2 });
	}

	vector<vector<long long>> dp(n + 1, vector<long long>(k + 1, -1));
	dp[0][0] = 0;

	for (int i = 1; i <= n; i++)
	{
		long walkCost = walk[i - 1].first;
		long walkValue = walk[i - 1].second;
		long cycleCost = cycle[i - 1].first;
		long cycleValue = cycle[i - 1].second;

		for (int j = 0; j <= k; j++)
		{
			long long best = -1;

			if (j >= walkCost &&
				dp[i - 1][j - walkCost] != -1)
			{
				best = max(best, dp[i - 1][j - walkCost] + walkValue);
			}

			if (j >= cycleCost &&
				dp[i - 1][j - cycleCost] != -1)
			{
				best = max(best, dp[i - 1][j - cycleCost] + cycleValue);
			}

			dp[i][j] = best;
		}
	}

	long long ans = 0;
	for (int i = 0; i <= k; i++)
	{
		ans = max(ans, dp[n][i]);
	}

	cout << ans;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/4412b885-a322-44f8-bf58-55da059d415e)](https://github.com/user-attachments/assets/4412b885-a322-44f8-bf58-55da059d415e){: .image-popup}<br>

솔직히 생각보다 너무 어려웠다<br>

- dp[i][j] 의 초기화를 '0'으로 하였기에<br>
  '도달 불가능' 상태를 체크하지 못하였음<br>

- dp[i][j]의 반복문을 '걷기' 와 '자전거'를 '따로' 구하였기에<br>
  자전거가 '동일 타이밍'에 비교되지 않아<br>
  걷기의 데이터를 덮어씌움<br>
  - 2차원 dp임에도 j 진행을 역순으로 진행하여<br>
    코드가 직관적이지 못하였음<br>

DP에 대한 초기 접근 방식 자체는 맞았으나<br>
'상태 전이'에 대한 구체적인 계획을 세우지 못하였고<br>
다음 상태로 넘어가는 부분에 대한 구현도 잘못되었었다<br>

