---
title: "백준 Gold 3 파일 합치기"
date : "2025-09-01 09:00:00 +0900"
last_modified_at: "2025-09-01T09:00:00"
categories:
  - 코딩 테스트
tags:
  - dp
---

## 파일 합치기 (백준 Gold 3)
<https://www.acmicpc.net/problem/11066><br>

연속된 파일을 합쳐서 하나의 파일을 만드는 데<br>
필요한 최소 비용을 구하는 문제<br>

- 파일 1 : 40, 파일 2 : 30 이면<br>
  합쳐진 파일 x = 40 + 30이다<br>
  여기까지의 비용은 70<br>

- 이후 파일 x와 파일 3 : 30을 합치면<br>
  파일 y = 70 + 30 = 100이지만<br>
  여기까지의 총 비용은 170<br>
  
이런식으로<br>
모든 연속된 파일들을 합치면서<br>
'총 비용'을 적게 만드는 것이 목표<br>

## 풀이 방식

일단, 이전 값을 활용할 수 있기에<br>
dp문제라 생각하였다<br>

또한 0~k 의 '특정 범위'에 해당하는 값을 이용하기에<br>
2차원 dp라 생각<br>

- dp[0][k] : 0~k까지 합칠때 발생한 최소 비용<br>

개인적으로 여기서 많이 삽질한 부분이 있었는데<br>
바로 '파일 자체의 합'과 '합치는 비용'을 함께 계산하려 했던것<br>

나는 dp[0][k] 에 0~k까지의 합도 포함했기에<br>
dp[0][0] : 0번째 파일의 값<br>
으로 설정하고 문제를 풀었다<br>

그랬더니 개념의 혼동이 발생하여<br>
파일끼리만 더한다던가, 이상한 배율을 적용한다던가 등의 오류를 범했다<br>

다시 생각해보니<br>
dp[0][0] 은 '파일 1개' 그 자체이며<br>
합치지 않았기에 dp[0][0] = 0 이 되어야 했다<br>

동시에 파일들의 '합'자체를 관리하는 별도의 관리 공간이 필요하였다<br>
그래서 sums라는 합용 2차원 배열을 이용하였다<br>
(누적합을 사용하면 1차원 배열이 되었겠지만<br>
삽질을 하느라 추가적인 응용을 스킵하였다)<br>

이후 파일을 합칠때<br>
dp[0][k]<br>
min(dp[0][0] + sums[0][0] + dp[1][k] + sums[1][k],<br>
dp[0][1] + sums[0][1] + dp[2][k] + sums[2][k] ...)<br>
방식으로 합쳐나가게 되었다<br>

## 제출 코드
```
#include<iostream>
#include<vector>
#include<limits.h>

using namespace std;

typedef unsigned long long ull;

int main()
{
	int t;
	cin >> t;
	while (t > 0)
	{
		t--;
		int n;
		cin >> n;

		vector<vector<ull>> sums(n, vector<ull>(n, LONG_MAX)); // 각 범위의 합들
		vector<vector<ull>> dp(n, vector<ull>(n, LONG_MAX)); // 합쳐지는 비용

		for (int i = 0; i < n; i++)
		{
			cin >> sums[i][i];
			dp[i][i] = 0;
		}

		for (int i = 0; i < n - 1; i++)
		{
			sums[i][i + 1] = sums[i][i] + sums[i + 1][i + 1];
			dp[i][i + 1] = sums[i][i + 1];
		}

		for (int size = 2; size < n; size++)
		{
			for (int i = 0; i < n - size; i++)
			{
				ull minV = LONG_MAX;
				sums[i][i + size] = sums[i][i] + sums[i + 1][i + size];
				
				for (int j = 0; j < size; j++)
				{
					ull temp1 = dp[i][i + j] + sums[i][i+j];
					ull temp2 = dp[i + j + 1][i + size] + sums[i+j+1][i+size];

					minV = min(minV, temp1 + temp2);
				}
				
				// dp[0][3]
				dp[i][i + size] = minV;
			}
		}

		cout << dp[0][n - 1] << '\n';
	}

	return 0;
}
```

## 결과
<img width="1151" height="120" alt="Image" src="https://github.com/user-attachments/assets/0d239ea6-81a7-43a5-9a17-def6820c76d0" /><br>

컴파일 오류는 unsigned 를 unsinged 로 잘못입력하여 발생한 오류이다<br>