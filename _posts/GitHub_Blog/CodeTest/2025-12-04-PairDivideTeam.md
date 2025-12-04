---
title: "백준 Gold 1 공평하게 팀 나누기"
date : "2025-12-04 10:30:00 +0900"
last_modified_at: "2025-12-04T10:30:00"
categories:
  - 코딩 테스트
tags:
  - DP
  - 다이나믹 프로그래밍
  - 배낭 문제
---

## 공평하게 팀 나누기 (백준 Gold 1)
<https://www.acmicpc.net/problem/4384><br>

N명의 인원수와 그 몸무게가 주어진다<br>
두 팀간의 인원수 차이가 1이하가 되며,<br>
몸무게 차이를 최소화하게 할 때<br>
두 팀의 무게를 오름차순으로 출력하는 문제<br>

## 풀이 방법

**dp 정의**<br>
- dp[i][j] : i 명일 때, j 몸무게가 가능한지를 체크<br>
  - i번째 사람을 체크하는게 아닌<br>
    '가능한 인원 수'의 체크<br>

- 한 팀의 몸무게를 구하게 되면<br>
  다른팀의 몸무게 = 총 몸무게 - 한 팀의 몸무게<br>

- 또한, 인원수가 1명 차이나는 부분도<br>
  n/2 + 1 로 해결 가능함<br>

**다음 상태 이동(점화식)**<br>
- dp[i][j] = dp[i-1][j-noww];
  이전 단계에서 해당 몸무게가 가능했다면 true인 방식<br>

**순회 방식**<br>
- 특이하게도, 인원수를 세는 count와<br>
  총 몸무게 j 역시 모두 내림차순을 적용해야 풀 수 있음<br>
  - count 내림차순?<br>
    : 같은 인원을 '다시 사용'하는 결과를 막기 위함<br>
  - 총 몸무게 내림차순?<br>
    : 역시 같은 몸무게를 '다시 사용'하는 결과를 막기 위함<br>


이후<br>
그렇게 정의된 dp[n/2]를 순회하며<br>

- abs(maxx - i * 2)가 가장 작은 i를 구한 후<br>
  i 와 maxx - i 를 구하면 문제를 풀 수 있음<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	int n;
	cin >> n;

	int maxx = 0;
	vector<int> weis(n);
	for (int i = 0; i < n; i++)
	{
		cin >> weis[i];
		maxx += weis[i];
	}

	vector<vector<bool>> dp(n / 2 + 1, vector<bool>(maxx + 1, false));
	dp[0][0] = true;

	for (int i = 0; i < n; i++)
	{
		int nowW = weis[i];
		for (int count = n / 2; count >= 1; count--)
		{
			for (int j = maxx; j >= nowW; j--)
			{
				if (dp[count - 1][j - nowW] == false)
					continue;

				dp[count][j] = true;
			}
		}
	}

	int minV = 1e9;
	pair<int, int> vv;

	for (int i = 0; i <= maxx; i++)
	{
		if (dp[n / 2][i] == false)
			continue;

		int v = abs(maxx - 2 * i);
		if (v < minV)
		{
			minV = v;

			int a = i;
			int b = maxx - i;

			if (a > b)
				swap(a, b);

			vv.first = a;
			vv.second = b;
		}
	}

	cout << vv.first << ' ' << vv.second;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/b16eda33-592d-49f6-836b-358d7e283a33)](https://github.com/user-attachments/assets/b16eda33-592d-49f6-836b-358d7e283a33){: .image-popup}<br>

count를 내림차순 할 생각을 제대로 하지 못하여 꽤 해맨 문제였다<br>

- 정확히는 nowW를 어떻게 활용할지 제대로 감을 잡지 못하였다<br>
  - dp 설계에 조금 응용이 들어가는 것이 까다로워지는 이유<br>
  - '이러면 될 것 같은데' '적용'을 어떻게 시키지...??<br>
