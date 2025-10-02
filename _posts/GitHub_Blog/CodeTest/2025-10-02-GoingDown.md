---
title: "백준 Gold 5 내려가기"
date : "2025-10-02 10:30:00 +0900"
last_modified_at: "2025-10-02T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 다이나믹 프로그래밍
---

## 내려가기 (백준 Gold 5)
<https://www.acmicpc.net/problem/2096><br>

N줄에 0~9의 숫자가 주어지고<br>
첫줄에서 끝줄까지 내려가며 밟은 칸의 숫자를 더한 합을 얻는다<br>
(다만 첫칸 -> 3번째 칸, 3번째칸 -> 첫칸 으로 2번 뛰어넘기는 x)<br>

주어지는 조건에서 얻을 수 있는 최대 합과 최소 합을 구하는 문제<br>

## 풀이 방법

기본적으로 DP계열 문제이다<br>
2개의 최대, 최소 에 관련된 dp 배열을 통해 문제를 구하면 풀 수 있다<br>

....<br>

그런데 메모리 제한이 C++ 기준으로 4MB다...?<br>

따라서 '한번 쓴' 데이터는 계속 저장하지 말고<br>
1줄 받은 데이터를 사용 후에<br>
뒤로 밀어버리는 작업을 통해 사용하는 메모리를 최소로 줄여야 한다!<br>

- 또한, Vector 대신 정적 배열을 사용하여 메모리 사용량을 더욱 줄였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<math.h>
#include<limits.h>

using namespace std;

int main()
{
	int n;
	cin >> n;
	int maps[3] = { 0, };
	int maxDp[2][3] = { 0, };
	int minDp[2][3];
	for (int i = 0; i < 2; i++)
	{
		for (int j = 0; j < 3; j++)
			minDp[i][j] = INT_MAX / 2;
	}

	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < 3; j++)
		{
			cin >> maps[j];

			if (i == 0)
			{
				maxDp[i][j] = maps[j];
				minDp[i][j] = maps[j];
			}
		}

		if (i > 0)
		{
			for (int j = 0; j < 3; j++)
			{
				if (j > 0)
				{
					maxDp[1][j - 1] = max(maxDp[1][j - 1], maxDp[0][j] + maps[j - 1]);
					minDp[1][j - 1] = min(minDp[1][j - 1], minDp[0][j] + maps[j - 1]);
				}

				maxDp[1][j] = max(maxDp[1][j], maxDp[0][j] + maps[j]);
				minDp[1][j] = min(minDp[1][j], minDp[0][j] + maps[j]);

				if (j < 2)
				{
					maxDp[1][j + 1] = max(maxDp[1][j + 1], maxDp[0][j] + maps[j + 1]);
					minDp[1][j + 1] = min(minDp[1][j + 1], minDp[0][j] + maps[j + 1]);
				}
			}

			for (int j = 0; j < 3; j++)
			{
				maxDp[0][j] = maxDp[1][j];
				maxDp[1][j] = 0;

				minDp[0][j] = minDp[1][j];
				minDp[1][j] = INT_MAX / 2;
			}
		}
		
	}

	int maxV = maxDp[0][0];
	int minV = minDp[0][0];

	for (int i = 1; i < 3; i++)
	{
		if (maxDp[0][i] > maxV)
			maxV = maxDp[0][i];

		if (minDp[0][i] < minV)
			minV = minDp[0][i];
	}

	cout << maxV << " " << minV;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/d823bbfd-27d1-49a1-870d-fdace89ba1d4)](https://github.com/user-attachments/assets/d823bbfd-27d1-49a1-870d-fdace89ba1d4){: .image-popup}<br>

최근에는 메모리를 넉넉하게 사용하는 문제만 풀었기에 잊고 있었지만<br>
예전에 배열을 사용하는 의미를 다시 떠올릴 수 있었던 좋은 문제였던 것 같다<br>