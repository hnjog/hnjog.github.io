---
title: "백준 Gold 4 리모컨"
date : "2026-01-12 12:00:00 +0900"
last_modified_at: "2026-01-12T12:00:00"
categories:
  - 코딩 테스트
tags:
  - 브루트포스
  - 재귀
---

## 리모컨 (백준 Gold 4)
<https://www.acmicpc.net/problem/1107><br>

이동 목표 채널과, 고장난 버튼의 수가 주어질 때<br>
목표 채널로 이동하기 위한 최소 명령횟수를 구하는 문제<br>

- 처음 시작 위치는 100<br>

- 리모컨의 버튼을 누르는 횟수도 포함<br>
  (ex : 1000 채널로 이동시, 4번의 버튼 입력 소요)<br>

- 기본적인 채널 더하기,빼기 이동 존재<br>
  - 101로 이동하려면 100에서 ++ 하면 1번만에 이동 가능<br>

## 풀이 방법  

dfs 문제처럼 보였지만<br>
결국 모든 가능성을 고려해야하는 문제였다<br>

1. 현재 시작 자릿수(100)에서 ++,--를 적용했을때의 횟수<br>
2. 목표 target보다 '한자릿수' 낮은, 만들 수 있는 수를 통해 적용 가능 횟수<br>
3. 목표 target보다 '한자릿수' 높은, 만들 수 있는 수를 통해 적용 가능 횟수<br>
4. 목표 target과 같은 자릿수에서, 만들 수 있는 수를 통해 적용 가능 횟수<br>

이와 같은 요소 중, 가장 작은 값을 가지는 요소를 구하면 풀 수 있는 문제<br>

## 제출 코드

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<limits.h>

using namespace std;

void getSeam(int target,const vector<bool>& buttons, int nowV, int nowC,const int baseC, int& ret)
{
	if (nowC == baseC)
	{
		if (nowC != 0)
			ret = min(ret, abs(target - nowV));
		return;
	}

	for (int i = 0; i < 10; i++)
	{
		if (buttons[i] == false)
			continue;

		getSeam(target, buttons, nowV * 10 + i, nowC + 1, baseC, ret);
	}
}

void getSeam2(int target, const vector<bool>& buttons, int nowV, int nowC, const int baseC, int& ret)
{
	if (nowC == baseC - 1)
	{
		if(nowC != 0)
			ret = min(ret, abs(target - nowV));
		return;
	}

	for (int i = 0; i < 10; i++)
	{
		if (buttons[i] == false)
			continue;

		getSeam2(target, buttons, nowV * 10 + i, nowC + 1, baseC, ret);
	}
}

void getSeam3(int target, const vector<bool>& buttons, int nowV, int nowC, const int baseC, int& ret)
{
	if (nowC == baseC + 1)
	{
		if (nowC != 0)
			ret = min(ret, abs(target - nowV));
		return;
	}

	for (int i = 0; i < 10; i++)
	{
		if (buttons[i] == false)
			continue;

		getSeam3(target, buttons, nowV * 10 + i, nowC + 1, baseC, ret);
	}
}

int main()
{
	int nowNum = 100;
	int target;
	cin >> target;

	vector<bool> buttons(10, true);
	int m;
	cin >> m;

	for (int i = 0; i < m; i++)
	{
		int t;
		cin >> t;
		buttons[t] = false;
	}

	int baseC = to_string(target).size();
	int ret1 = 1e9;
	int ret2 = 1e9;
	int ret3 = 1e9;

	getSeam(target, buttons, 0, 0, baseC, ret1);
	getSeam2(target, buttons, 0, 0, baseC, ret2);
	getSeam3(target, buttons, 0, 0, baseC, ret3);

	int bV = abs(100 - target);
	int sv1 = ret1 + baseC;
	int sv2 = ret2 + baseC - 1;
	int sv3 = ret3 + baseC + 1;

	int v = min(bV, sv1);
	v = min(sv1, v);
	v = min(sv2, v);
	v = min(sv3, v);

	cout << v;

	return 0;
}
```

## 결과

[![Image](https://github.com/user-attachments/assets/52c89b5c-93ba-42a4-b185-071da82d2c95)](https://github.com/user-attachments/assets/52c89b5c-93ba-42a4-b185-071da82d2c95){: .image-popup}<br>

브루트 포스는 항상 구현력을 시험하는 듯한 문제가 많은 듯하다<br>
