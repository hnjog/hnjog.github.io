---
title: "백준 Gold 3 탑 보기"
date : "2025-09-05 11:00:00 +0900"
last_modified_at: "2025-09-05T11:00:00"
categories:
  - 코딩 테스트
tags:
  - 스택
---

## 탑 보기 (백준 Gold 3)
<https://www.acmicpc.net/problem/22866><br>

'일직선'으로 다양한 높이의 건물이 N개 주어진다<br>
'각각의 건물 옥상'에서 양 옆의 위를 바라보았을때<br>
모든 건물들의<br>
보이는 총 건물 개수와 '가장 가까이 보이는 건물'의 인덱스를 구하는 문제<br>

## 접근 방식

건물 간의 사이는 총 3가지로 나뉜다<br>
a가 본 '오른쪽 건물의 가장 큰 높이', b가 현재 건물의 높이<br>
(오른쪽 건물 가장 큰 높이 의 초기값은 자신 건물의 높이)<br>

- a < b<br>
 : a의 '볼 수 있는 건물 개수'를 + 1<br>
   이후, a 입장에서 '오른쪽' 건물 높이를<br>
   b로 조정하고, b가 가장 가까운 건물인지 검사<br>

- a == b <br>
 : a가 가지고 있는 '보이는 건물 개수'를 카피<br>
   (a에서 보인 건물은 b에서도 보이므로)<br>
   이후 break<br>
   (a가 보인 건물이 곧 b가 볼 수 있는 건물이므로)<br>
   (또한 a가 가지는 '가장 가까운 건물'도 거리 +1 해서 가지고 옴)<br>

- a > b <br>
 : b의 가장 가까운 건물을 a로 지정<br>
 (b 입장에선 -1인 가장 가까이 보이는 건물이므로)<br>
 b의 건물 개수 + a의 건물 개수 + 1<br>
 이후 break<br>
 (마찬가지로 a가 볼 수 있는 건물은 b도 볼 수 있음)<br>

해당 건물 비교를<br>
1 -> n까지 반복하는 방식<br>

## 첫 제출 코드

```cpp
#include<iostream>
#include<stack>
#include<vector>
#include<unordered_set>
#include<limits.h>

using namespace std;

struct tInfo
{
	int myIdx = 0;
	int myHeight = 0;
	int bestIdx = 0; // 보이는 것 중 가장 가까운 인덱스
	int bestLength = INT_MAX; // bestIdx를 정하기 위한 길이 abs(비교Idx - myIdx)
	int bestLeftHeight = 0; // 만난 최고 높이 건물
	int bestRightHeight = 0; // 만난 최고 높이 건물

	unordered_set<int> lessIdx;
	unordered_set<int> higherIdx;
	unordered_set<int> sameIdx;
};

void check(vector<tInfo>& tInfos, tInfo& now)
{
	if (now.myIdx == 0)
		return;

	int nowIdx = now.myIdx;
	int nowHeight = now.myHeight;

	for (int i = nowIdx - 1; i >= 1; i--)
	{
		tInfo& tempI = tInfos[i];

		if (tempI.bestRightHeight < nowHeight)
		{
			tempI.bestRightHeight = nowHeight;
			tempI.higherIdx.insert(nowIdx);

			int len = nowIdx - i;
			if (tempI.bestLength > len)
			{
				tempI.bestIdx = nowIdx;
				tempI.bestLength = len;
			}

			now.lessIdx.insert(i);

			for (int l : tempI.lessIdx)
			{
				tInfos[l].higherIdx.insert(nowIdx);
			}

			for (int s : tempI.sameIdx)
			{
				tInfos[s].higherIdx.insert(nowIdx);
			}

			continue;
		}

		if (tempI.myHeight > nowHeight)
		{
			tempI.lessIdx.insert(nowIdx);

			int len = nowIdx - i;
			if (now.bestLength > len)
			{
				now.bestIdx = i;
				now.bestLength = len;
			}

			if(now.bestLeftHeight < tempI.myHeight)
				now.bestLeftHeight = tempI.myHeight;

			now.higherIdx.insert(i);

			for (int h : tempI.higherIdx)
			{
				now.higherIdx.insert(h);
			}

			break;
		}
		
		
		now.sameIdx.insert(i);

		if (tempI.bestIdx != 0)
		{
			now.higherIdx = tempI.higherIdx;
			now.bestIdx = tempI.bestIdx;
			now.bestLength = tempI.bestLength + (nowIdx - tempI.myIdx);
		}
		
		for(int s : tempI.sameIdx)
			now.sameIdx.insert(s);

		break;
	}
}

int main()
{
	int n;
	cin >> n;
	vector<tInfo> tInfos(n+1);

	for (int i = 0; i < n; i++)
	{
		tInfos[i + 1].myIdx = i + 1;
		cin >> tInfos[i + 1].myHeight;
		tInfos[i + 1].bestLeftHeight = tInfos[i + 1].myHeight;
		tInfos[i + 1].bestRightHeight = tInfos[i + 1].myHeight;
	}

	for (int i = 1; i <= n; i++)
	{
		check(tInfos, tInfos[i]);
	}

	for (int i = 1; i <= n; i++)
	{
		cout << tInfos[i].higherIdx.size();

		if (tInfos[i].higherIdx.size() != 0)
			cout << " " << tInfos[i].bestIdx;

		cout << '\n';
	}

	return 0;
}
```

## 틀린 이유

각각의 map이 '넓어지면서'<br>
안그래도 100001개 까지 보관하고 있는 스택이 터짐<br>
(메모리 초과)<br>

## 수정 방법
각 map들에 대한 내용을 제거하고<br>
'보이는 건물 개수'를 추가<br>
'각 건물들 사이의 관계'를 정리하여 해당 개수를 조정<br>

## 최종 제출 코드

```cpp
#include<iostream>
#include<stack>
#include<vector>
#include<unordered_set>
#include<limits.h>

using namespace std;

struct tInfo
{
	int myIdx = 0;
	int myHeight = 0;
	int bestIdx = 0; // 보이는 것 중 가장 가까운 인덱스
	int bestLength = INT_MAX; // bestIdx를 정하기 위한 길이 abs(비교Idx - myIdx)
	int bestRightHeight = 0; // 만난 최고 높이 건물
	int higherCount = 0;
};

void check(vector<tInfo>& tInfos, tInfo& now)
{
	if (now.myIdx == 0)
		return;

	int nowIdx = now.myIdx;
	int nowHeight = now.myHeight;

	for (int i = nowIdx - 1; i >= 1; i--)
	{
		tInfo& tempI = tInfos[i];

		if (tempI.bestRightHeight < nowHeight)
		{
			tempI.bestRightHeight = nowHeight;
			tempI.higherCount++;

			int len = nowIdx - i;
			if (tempI.bestLength > len)
			{
				tempI.bestIdx = nowIdx;
				tempI.bestLength = len;
			}

			continue;
		}

		if (tempI.myHeight > nowHeight)
		{
			int len = nowIdx - i;
			if (now.bestLength > len)
			{
				now.bestIdx = i;
				now.bestLength = len;
			}

			now.higherCount += tempI.higherCount;
			now.higherCount++;
			break;
		}
		
		now.higherCount = tempI.higherCount;
		if (tempI.bestIdx != 0)
		{
			now.bestIdx = tempI.bestIdx;
			now.bestLength = tempI.bestLength + (nowIdx - tempI.myIdx);
		}

		break;
	}
}

int main()
{
	int n;
	cin >> n;
	vector<tInfo> tInfos(n+1);

	for (int i = 0; i < n; i++)
	{
		tInfos[i + 1].myIdx = i + 1;
		cin >> tInfos[i + 1].myHeight;
		tInfos[i + 1].bestRightHeight = tInfos[i + 1].myHeight;
	}

	for (int i = 1; i <= n; i++)
	{
		check(tInfos, tInfos[i]);
	}

	for (int i = 1; i <= n; i++)
	{
		cout << tInfos[i].higherCount;

		if (tInfos[i].higherCount != 0)
			cout << " " << tInfos[i].bestIdx;

		cout << '\n';
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/ec0ad808-66b9-4bf7-b223-baf26153214d)](https://github.com/user-attachments/assets/ec0ad808-66b9-4bf7-b223-baf26153214d){: .image-popup}<br>

구현이 생각보다 어려웠다<br>
또 스택의 개념을 사용하긴 하지만<br>
(바로 이전 건물 체크)<br>

a < b 부분에 대한 스택을 적용하기 어려워<br>
반복문으로 구하였다<br>
('이전 건물'들이 현재 높이를 확인해야 한다고 생각했다)<br>