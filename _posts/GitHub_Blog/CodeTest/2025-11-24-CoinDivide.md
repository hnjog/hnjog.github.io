---
title: "백준 Gold 2 동전 분배"
date : "2025-11-24 10:30:00 +0900"
last_modified_at: "2025-11-24T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 동전 분배 (백준 Gold 2)
<https://www.acmicpc.net/problem/1943><br>

3개의 입력이 주어지고<br>
동전의 종류 N이 주어진다<br>

각 동전의 개수와 가치가 주어질 때<br>
동전의 가치를 절반으로 나눌수 있는지 여부를 출력하는 문제<br>

- 나눌 수 없다면 0<br>
  나눌 수 있다면 1 출력<br>

## 풀이 방법

먼저 접근한 방식은 다음과 같다<br>

1. 총 가치 MaxV를 잡음<br>
2. 만약 MaxV가 홀수라면 '나눌 수 없음'이므로 0 출력<br>
3. 아니라면 '절반'의 수치인 TargetV = MaxV / 2 를 잡는다<br>
4. 이후 dp를 돌면서 TargetV를 '만들 수 있는지'를 체크<br>

- 특이한 점으로<br>
  '어느 시점'이라도 dp[..][TargetV] 를 만들 수 있다면<br>
  결과적으로 정답임<br>

- **'모든 동전의 총합'의 '절반'을 '만들 수 있다'**는 점에서<br>
   **남아있는 동전의 합이 '절반'**이 되므로<br>
   더 이상 연산을 할 필요가 없음<br>
   (끝까지 연산하면 '시간 초과' 발생)<br>

**점화식**<br>
- dp[i][j]<br>
 : i번째 동전을 주어진 만큼 사용하였을 때,<br>
   j 값을 만들 수 있는지의 여부<br>

**다음 상태 이동**<br>
- 동전의 '개수'가 주어지므로<br>
  그 개수만큼 반복문을 돌면서 '해당 수치'가 가능한지를 파악<br>
  dp[i][j] = dp[i-1][j - k * v] (0 <= k <= 해당 동전 횟수)<br>
  (이전 상태에서 해당 수치를 빼었을때 true이면 현재 값을 만들수 있다는 뜻)<br>

- k가 반복문을 돌려야 하기에 <br>
  O(n^3)이기에 끝까지 돌리면 안되는 이유이기도 하다<br>

**순회 방식**<br>
- 2차원 dp이며, j가 '값'을 표현하는 방식이기에<br>
  오름차순/내림차순이 크게 영향 없는 편<br>

- 다만 1차원 dp로 바꿀 수 있는 가능성이 있음<br>
  + 그 경우엔 j를 내림차순으로 하는 것을 권장<br>
  (dp 조건이 바뀜)<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	ios::sync_with_stdio(false);
	cin.tie(nullptr);

	int idx = 0;

	while (idx < 3)
	{
		int n;
		cin >> n;
		vector<int> coins;
		vector<int> counts;

		int maxV = 0;

		for (int i = 0; i < n; i++)
		{
			int v, c;
			cin >> v >> c;

			coins.push_back(v);
			counts.push_back(c);

			for (int j = 0; j < c; j++)
			{
				maxV += v;
			}
		}

		if (maxV % 2 == 1)
		{
			cout << 0 << '\n';
			idx++;
			continue;
		}

		// n개의 동전 사용하여 총 합의 절반인 targetV를 표현할 수 있는가?
		int targetV = maxV / 2;
		vector<vector<bool>> dp(n + 1, vector<bool>(targetV + 1, false));

		dp[0][0] = true;

		bool bFind = false;

		for (int i = 1; i <= n; i++)
		{
			int nowCoinV = coins[i - 1];
			int nowCoinC = counts[i - 1];

			for (int j = targetV; j >= 0; j--)
			{
				for (int k = nowCoinC; k >= 0; k--)
				{
					int tj = j - k * nowCoinV;

					if (tj >= 0 &&
						dp[i-1][tj])
					{
						dp[i][j] = true;
						if (j == targetV)
							bFind = true;
						break;
					}
				}

				if (bFind)
					break;
			}
			if (bFind)
				break;
		}

		if (bFind)
			cout << 1 << '\n';
		else
			cout << 0 << '\n';

		idx++;
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/d271ca3d-9118-4d68-8b1f-7be9601c5b74)](https://github.com/user-attachments/assets/d271ca3d-9118-4d68-8b1f-7be9601c5b74){: .image-popup}<br>


토요일부터 꾸준히 풀면서<br>
계속 힌트를 받고, '이해'하는게 참 힘들었던 문제이다<br>

- 처음에는 dp[n][0]을 만들수 있는지를 체크하다가<br>
  메모리 초과 + 시간 초과를 받아서<br>
  dp 조건을 바꾸었다<br>
  (이전에 풀었던 저울 문제처럼 풀다가 포기)<br>

dp에 좀 익숙해진 줄 알았지만<br>
여전히 '조건'을 잡는데 애를 먹고 있다<br>

- 사실 정답 코드를 보면 이해하기 쉽지만<br>
  정작 내가 짜려고 하면, 어떻게 짜야할지 정말 막막하다<br>
  - 특히 특정 dp 조건에 대한 접근이 '실패'하였을 때<br>
    다시 dp 조건을 '어떻게' 짜야 할지 생각하기 힘들다<br>
