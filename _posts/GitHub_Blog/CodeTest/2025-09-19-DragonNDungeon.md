---
title: "백준 Gold 4 드래곤 앤 던전"
date : "2025-09-19 12:30:00 +0900"
last_modified_at: "2025-09-19T12:30:00"
categories:
  - 코딩 테스트
tags:
  - 구현
  - 이분 탐색
---

## 드래곤 앤 던전 (백준 Gold 4)
<https://www.acmicpc.net/problem/16434><br>

용사가 주어지는 던전을 클리어하기 위하여<br>
최대 체력이 1 늘어나는 수련을 '몇 번' 진행하는지에 대한 문제<br>

## 풀이 방법

요점은 '용사의 수련 횟수'를 최소한으로 하며 클리어 하는 것<br>
따라서 해당 조건에 대하여 이분 탐색을 진행할 수 있다<br>

- '용사'의 수련 횟수를 mid 값으로 잡은 후<br>
  해당 값으로 던전을 통과 가능한지를 테스트<br>

- 통과 못하면 left를 mid + 1,<br>
  통과 했다면 right를 mid로<br>
  (일단 통과 가능한 값을 right로 잡아 아예 못찾는 경우 배제)<br>

## 첫 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<limits.h>

using namespace std;

int n;
long long heroBaseAttack;

bool isSuccess(vector<vector<long long>>& dInfos, long long heroBaseHp)
{
	long long currentHp = heroBaseHp;
	long long currentAttack = heroBaseAttack;

	for (auto& room : dInfos)
	{
		int type = room[0];

		if (type == 1)
		{
			int eAt = room[1];
			int eHp = room[2];

			while (true)
			{
				eHp -= currentAttack;
				if (eHp <= 0)
					break;

				currentHp -= eAt;
				if (currentHp <= 0)
					return false;
			}
		}
		else
		{
			currentAttack += room[1];
			currentHp += room[2];
			currentHp = min(currentHp, heroBaseHp);
		}
	}

	return true;
}

int main()
{
	cin >> n;
	cin >> heroBaseAttack;

	vector<vector<long long>> dInfos(n, vector<long long>(3));

	for (int i = 0; i < n; i++)
	{
		cin >> dInfos[i][0];
		cin >> dInfos[i][1];
		cin >> dInfos[i][2];
	}

	long long left = 0;
	long long right = LLONG_MAX / 2;

	while (left < right)
	{
		long long mid = (left + right) / 2;

		if (isSuccess(dInfos, mid))
		{
			right = mid;
		}
		else
		{
			left = mid + 1;
		}
	}

	cout << left;

	return 0;
}
```

## 틀린 이유

총 2가지 부분에 대하여 틀렸었다<br>

- 전투 부분<br>
  : 해당 코드를 더 간략하게 만들 수 있다<br>
   용사는 t = (적의 체력 / 용사의 공격력) 횟수 만큼 타격하므로<br>
   굳이 반복할 필요가 없음<br>
   또한 적의 반격은 t-1 번 하므로<br>
   '적이 죽는지 여부'에 상관없이<br>
   전투가 끝난 시점에 용사의 체력이 0보다 작은지를 판별하면 됨<br>
   (while 로 냅두면 시간초과 발생)<br>

- 오버플로우 부분<br>
  : hp를 회복하는 부분에서<br>
    (currentHp + room[2])<br>
	해당 부분에서 오버플로우가 발생하여<br>
	current 값이 올바르지 않게 들어갈 가능성이 있었다<br>
	따라서 해당 부분을 좀 더 면밀히 수정하였다<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<limits.h>
#include<math.h>

using namespace std;

int n;
long long heroBaseAttack;

bool isSuccess(vector<vector<long long>>& dInfos, long long heroBaseHp)
{
	long long currentHp = heroBaseHp;
	long long currentAttack = heroBaseAttack;

	for (auto& room : dInfos)
	{
		int type = room[0];

		if (type == 1)
		{
			long long eAt = room[1];
			long long eHp = room[2];

			long long t = ceil(eHp / (long double)currentAttack);
			if (currentHp <= (t - 1) * eAt)
				return false;

			currentHp -= (t - 1) * eAt;
		}
		else
		{
			currentAttack += room[1];
			
			if (currentHp < heroBaseHp)
			{
				long long amount = heroBaseHp - currentHp;
				if (room[2] >= amount)
					currentHp = heroBaseHp;
				else
					currentHp += room[2];
			}

		}
	}

	return true;
}

int main()
{
	cin >> n;
	cin >> heroBaseAttack;

	vector<vector<long long>> dInfos(n, vector<long long>(3));

	for (int i = 0; i < n; i++)
	{
		cin >> dInfos[i][0];
		cin >> dInfos[i][1];
		cin >> dInfos[i][2];
	}

	long long left = 1;
	long long right = LLONG_MAX;

	while (left < right)
	{
		long long mid = (left + right) / 2;

		if (isSuccess(dInfos, mid))
		{
			right = mid;
		}
		else
		{
			left = mid + 1;
		}
	}

	cout << left;

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/9720d1ab-f61c-4c06-a3df-53a3b19590af)](https://github.com/user-attachments/assets/9720d1ab-f61c-4c06-a3df-53a3b19590af){: .image-popup}<br>

분명 '자료형'만 관련된 문제인 줄 알았으나<br>
여러 형태로 바꾸어도 개선이 되지 않자<br>
그제서야 로직을 보게 되었다<br>

구현에 조금 더 신경을 써야 했던 것 같다<br>
