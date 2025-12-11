---
title: "백준 Gold 3 세 용액"
date : "2025-12-11 10:30:00 +0900"
last_modified_at: "2025-12-11T10:30:00"
categories:
  - 코딩 테스트
tags:
  - 이분 탐색
  - 이진 탐색
---

## 세 용액 (백준 Gold 3)
<https://www.acmicpc.net/problem/2473><br>

임의의 특성값이 주어진 배열이 주어질 때<br>
3가지 요소를 골라 0에 가까운 조합을 만드는 문제<br>

- 0에 가장 가까운 조합이 여러개라면<br>
  그 중 아무거나 출력<br>

## 풀이 방법  

- 주어진 요소들을 '오름차순' 정렬<br>

- i를 기준으로 한칸씩 진행<br>
  - left : i + 1<br>
  - right : n - 1<br>
  - 이후 Sum = arr[i] + arr[left] + arr[right]를 구한 후<br>
    그 절댓값이 작은 값을 결과 배열에 저장<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include <climits>

using namespace std;

int main()
{
	int n;
	cin >> n;

	vector<long long> arr(n);
	for (int i = 0; i < n; i++)
		cin >> arr[i];

	sort(arr.begin(), arr.end());

	vector<long long> ret;
	long long dif = LONG_MAX;

	bool bFindz = false;

	for (int i = 0; i < n - 2; i++)
	{
		long long v1 = arr[i];
		int left = i + 1, right = n - 1;

		while (left < right)
		{
			long long v2 = arr[left];
			long long v3 = arr[right];
			long long d = (v1 + v2 + v3);

			if (dif > abs(d))
			{
				dif = abs(d);
				ret.clear();
				ret.push_back(v1);
				ret.push_back(v3);
				ret.push_back(v2);

				if (dif == 0)
				{
					bFindz = true;
					break;
				}
			}

			if (d < 0)
				left++;
			else
				right--;
		}

		if (bFindz)
			break;
	}

	sort(ret.begin(), ret.end());

	for (long long r : ret)
	{
		cout << r << ' ';
	}

	return 0;
}
```

## 결과
[![Image](https://github.com/user-attachments/assets/5eee9c39-49ac-4a1e-b5f4-f699fa35cb30)](https://github.com/user-attachments/assets/5eee9c39-49ac-4a1e-b5f4-f699fa35cb30){: .image-popup}<br>

처음에는 이분탐색을 '어디'에 적용해야 할지 몰랐다<br>

- i + j를 기반으로 이분탐색을 진행하려 했음<br>

그러나 적절한 풀이가 나오지 않으며<br>
예외처리가 까다로워 졌기에 접근 방식에 대하여 고민하였다<br>

- 기준점 i를 놓은 후<br>
  left , right를 움직이면서 '합'을 찾는 방식<br>
  (기준점을 냅두고 비교점을 움직이기에 '투 포인터'와 비슷할지도?)<br>

해당 방식을 찾아 로직을 고치니 통과할 수 있었다<br>