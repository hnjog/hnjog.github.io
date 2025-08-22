---
title: "백준 Silver 1 오르막수"
date : "2025-08-23 09:00:00 +0900"
last_modified_at: "2025-08-23T09:00:00"
categories:
  - 코딩 테스트
tags:
  - DP
---

## 오르막수 (백준 Silver 1)
<https://www.acmicpc.net/problem/11057><br>

오르막 수는 '수'의 자리가 오름차순을 이루는 수이다<br>
ex) 2234,3678 등<br>
반대로 2232,3676 같은 수는 오르막 수가 아니다<br>

N의 자릿수가 주어졌을 때<br>
만들 수 있는 오르막 수의 개수를 구하는 문제<br>
(수는 0으로 시작 가능)<br>

### 풀이 방식
일단 생각해볼 점<br>
dp[1] = 10<br>
-> 0~9 까지의 각 수가 1자리로 놓여있으면 각각 1로 센다<br>

dp[2] = 55<br>
그렇다면 이 경우는?<br>

```
0 - 0
1 - 1 0
2 - 2 1 0
...
9 - 9 8 7 6 5 ... 0

```

대충 규칙이 보이는 듯 하다<br>

dp[3] = 220<br>

```
0 - 00
1 - 00 01 10 11 (dp[2]의 0과 1의 합)
...

```

따라서 우리는 2차원 배열을 만들어야 한다<br>

dp[n][m] : n번째 자릿수에서 숫자 m이 만들 수 있는 오르막 수 개수<br>

그리고 마지막 dp[n] 번째의<br>
모든 자릿수를 더하면 정답을 구할 수 있다!<br>


## 제출 코드

```
#include<iostream>
#include<vector>

using namespace std;

const int divV = 10007;

int main()
{
	int n;
	cin >> n;
	vector<vector<int>> dp(n, vector<int>(10, 0));
	for (int i = 0; i < 10; i++)
	{
		dp[0][i] = 1;
	}

	for (int i = 1; i < n; i++)
	{
		for (int j = 0; j < 10; j++)
		{
			for (int k = 0; k <= j; k++)
			{
				dp[i][j] += (dp[i - 1][k] % divV);
				dp[i][j] %= divV;
			}
		}
	}

	int sum = 0;
	for (int i = 0; i < 10; i++)
	{
		sum += dp[n - 1][i];
		sum %= divV;
	}

	cout << sum;

	return 0;
}
```

## 결과
<img width="1161" height="159" alt="Image" src="https://github.com/user-attachments/assets/c2ca68d9-67cb-4efe-bc0a-497ee0388515" /><br>

2차원 dp를 고려하여 충분히 풀 수 있는 문제였다<br>
이전에는 점화식을 잘 세우지 못해 dp만 보면 끙끙되었는데<br>
조금씩 늘어가는 걸 체감하고 있다<br>