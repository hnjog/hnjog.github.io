---
title: "백준 Gold 4 함께 블록 쌓기"
date : "2025-11-25 10:30:00 +0900"
last_modified_at: "2025-11-25T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 함께 블록 쌓기 (백준 Gold 4)
<https://www.acmicpc.net/problem/18427><br>

n번 학생까지 최대 M까지, 높이가 각각 다른 블록들을 가지고 있다<br>
각 학생들이 블럭을 1개씩만 쌓을 수 있을 때<br>
h 높이가 되도록 할 수 있는 경우의 수를 구하는 문제<br>

## 풀이 방법

dp문제이기에 각 진행 방식을 검토하여 풀어보았다<br>

**점화식**<br>
- dp[i][j] : i번째 사람까지 만들 수 있는 j 값의 개수<br>

- 경우의 수를 세어야 하므로 int 를 사용하였다<br>

- 1차원 dp로는 '이전 상태'를 파악하기 힘들었기에<br>
  2차원 dp 사용<br>
  - 1차원 dp는 보통 '바로 직전'의 상태를 이용하여 문제를 풀 수 있는 경우에 사용<br>
  - 이 경우는 '다양한 높이값'을 합쳐 'h'를 만들어야 하므로 제외함<br>

**다음 상태 이동**<br>
- 이전 상태인 dp[i-1][j]의 값을 현재 상태에 포함시켜야 함<br>
  dp[i][j] += dp[i-1][j]<br>

- 또한 현재 i번째 사람이 들고 있는 블록의 개수를 통해<br>
  만들 수 있는 값들도, 이러한 경우의 수에 포함<br>
  dp[i][j + blocks[k]] += dp[i-1][j]<br>
  - ex) 1,2...n번째 까지 만들 수 있는 3의 개수가 3개라면<br>
        n+1 이 2 높이의 블록을 가진다면 dp[n+1][5] 에 3이 더해져야 함<br>

**순회 방식**<br>
- j 기준 오름차순 진행<br>
  (차순은 그렇게 상관은 없어보이긴 하다)<br>

- 블록을 순차적으로 탐색해야 하기에<br>
  O(N^3)으로 진행<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include <sstream>

using namespace std;

const int divV = 10007;

int main()
{
	string l;
	getline(cin, l);
	stringstream starts(l);
	int n, m,h;
	starts >> n;
	starts >> m;
	starts >> h;

	vector<vector<int>> blocks(n, vector<int>());

	for (int i = 0; i < n; i++)
	{
		getline(cin, l);
		stringstream ss(l);

		int t;
		while (ss >> t)
		{
			blocks[i].push_back(t);
		}
	}

	// dp[i][j] : i번째의 사람까지 만들 수 있는 j값의 경우의 수
	// dp[i-1][j]를 사실상 가져와야 함 (+=)
	vector<vector<int>> dp(n + 1, vector<int>(h + 1, 0));

	dp[0][0] = 1;

	for (int i = 1; i <= n; i++)
	{
		vector<int>& nowBlocks = blocks[i - 1];
		int nSize = nowBlocks.size();

		for (int j = 0; j <= h; j++)
		{
			if (dp[i - 1][j] <= 0)
				continue;

			dp[i][j] += dp[i - 1][j];
			dp[i][j] %= divV;

			for (int nV : nowBlocks)
			{
				if (j + nV > h)
					continue;

				dp[i][j + nV] += dp[i - 1][j];
				dp[i][j + nV] %= divV;
			}
		}
	}

	cout << dp[n][h] % divV;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/1eec853b-cede-47b0-a654-1ebb5855e5bf)](https://github.com/user-attachments/assets/1eec853b-cede-47b0-a654-1ebb5855e5bf){: .image-popup}<br>

난이도가 그리 높지는 않지만<br>
dp 문제를 푸는 방식에 익숙해져서 조금 다행이다 싶었다<br>

### TMI : dp 가 어려운 이유

개인적인 생각이나 dp는 다음과 같이 풀어진다고 생각한다<br>

1. 해당 문제가 dp인지 확인(실전)<br>
2. dp의 상태 정의하기 (1차원,2차원... 해당 dp의 의미)<br>
3. dp의 로직 검사 (상태 전이)
4. 예제 테스트 및 반례 검사
5. 만약 틀렸다면 다시 2번으로...

dp를 풀때는 항상 'dp 정의 -> 점화식 -> 코드 구현' 방식을 따르는 것이<br>
그나마 쉽게 풀 수 있다<br>
(+ 때로는 관점을 뒤집어보거나, 다르게 보면 문제가 쉽게 풀리는 경우도 있다)<br>

그렇기에 dp는 '프로그래머'가 문제를 대하는 자세와 가까운 느낌이다<br>

- 특정 문제에 대한 해결 관점<br>
- 그 관점으로 문제를 어떻게 풀지 로직짜기<br>
- 로직 구현<br>
- 다양한 테스트<br>
  - Happy Path : 머릿속에서 상상한 대로 잘 가겠지~ 라는 방식의 안일한 디버깅<br>
  - 실제로 '다양한 검증'과 상황을 통해 최소한의 구현 여부를 파악해야 함<br>
- 결과 체크 및 새로운 로직을 짜거나, 관점을 다르게 볼 수 있는지<br>

실제로는<br>
'새로운 로직'을 짜는 부분은 너무나 고통스럽고 시간이 많이 걸리기에<br>
보통은 이미 만든 결과물에서 타협을 하는 것이 종종 발생한다<br>

그러나 DP는 그러한 부분을 용납치 않기에<br>
문제를 풀 때 더욱 고통스러우며, '틀리는 것 조차' 하나의<br>
학습 과정이라 볼 수 있는것 같다<br>

- 그 문제를 틀릴 수록 '패턴화'되어<br>
  경험이 누적되기에<br>
