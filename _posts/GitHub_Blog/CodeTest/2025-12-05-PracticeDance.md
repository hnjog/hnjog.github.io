---
title: "백준 Gold 4 조교의 맹연습"
date : "2025-12-05 10:30:00 +0900"
last_modified_at: "2025-12-05T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 조교의 맹연습 (백준 Gold 4)
<https://www.acmicpc.net/problem/27114><br>

3가지 제식이 주어졌을 때<br>
제식 연습 비용이 3개, 에너지 k가 주어졌을때<br>

처음에 제자리로 돌아오기 위한 최소 연습 횟수를 구하는 문제<br>

- 각각 '좌','우','180도' 로 회전하는 연습 비용이 주어짐<br>
- 처음에 정면을 바라보고 있다고 가정<br>
- 정면으로 돌아올 수 없는 상황이라면 -1 출력<br>

## 풀이 방법

**dp 정의**<br>
- dp[i][j] : i번째 에너지를 소모하여 j 방향을 바라보는 최소 연습 횟수<br>

- 소모한 i번째 에너지와 '방향' 모두 관리해야 하기에 2차원 dp로 지정<br>

**다음 상태 이동(점화식)**<br>
- dp[j + noww][nextDir] = min(dp[j][dir] + 1, dp[j + noww][nextDir]);<br>

- 현재 비용을 넣을 수 있다면, 그 다음 방향에<br>
  횟수를 더해 넣어줌<br>

**순회 방식**<br>
- 오름차순!<br>
  - 연습 횟수에 대한 제한이 없기에<br>
    에너지만 충분하면 같은 방향으로 계속 연습 가능함<br>


```cpp
#include<iostream>
#include<vector>
#include<algorithm>

using namespace std;

int getdir(int dir, int order)
{
	dir += order;

	if (dir < 0)
		dir = 4 + dir;
	if (dir > 3)
		dir -= 4;

	return dir;
}

int main()
{
	vector<int> weights(3);
	for (int i = 0; i < 3; i++)
		cin >> weights[i];

	int dir[3] = { 1,-1,2 };

	int k;
	cin >> k;

	vector<vector<int>> dp(k + 1, vector<int>(4, -1));
	dp[0][0] = 0;

	for (int i = 0; i < 3; i++)
	{
		int noww = weights[i];
		int nowOrder = dir[i];

		for (int j = 0; j <= k;j++)
		{
			if (j + noww > k)
				break;

			for (int dir = 0; dir <= 3; dir++)
			{
				if (dp[j][dir] == -1)
					continue;

				int nextDir = getdir(dir, nowOrder);

				if (dp[j + noww][nextDir] == -1)
					dp[j + noww][nextDir] = dp[j][dir] + 1;
				else
					dp[j + noww][nextDir] = min(dp[j][dir] + 1, dp[j + noww][nextDir]);
			}
		}
	}

	cout << dp[k][0];

	return 0;
}
```

## 제출 코드

## 결과
[![Image](https://github.com/user-attachments/assets/b16eda33-592d-49f6-836b-358d7e283a33)](https://github.com/user-attachments/assets/b16eda33-592d-49f6-836b-358d7e283a33){: .image-popup}<br>

처음에 틀린것은 < 0에 대한 방향 처리를 잘못 잡았기 때문이었다<br>
(음수값이 확실한데 - 를 해버려서 dir 값이 이상해졌다)<br>