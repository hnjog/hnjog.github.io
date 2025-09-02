---
title: "백준 Gold 3 크게 만들기"
date : "2025-09-03 09:00:00 +0900"
last_modified_at: "2025-09-03T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 스택
  - 그리디 알고리즘
---

## 크게 만들기 (백준 Gold 3)
<https://www.acmicpc.net/problem/2812><br>

N자리 숫자가 주어졌을때,<br>
숫자 K개를 지워 얻을 수 있는 가장 큰 수를 구하는 문제<br>
(k는 n보다 작음)<br>

## 풀이 방법

당연한 얘기지만<br>
기본적인 자릿수는 N-K가 될 것이고<br>
결국 '맨' 앞에 N-K까지의 가능한 '큰 수'가 도달해야 가장 클 것이다<br>

(1924에서 2를 지운다고 했을때, 적어도 9가 맨 앞자리에 있어야 가장 클것이므로)<br>

그렇기에 결과용 string에 현재 값을 '담은 후'<br>
그 다음값과 비교하여 '작다면' k를 소모하여<br>

앞 자리를 제거하는 방식을 취하였다<br>

그런 방식으로 문자열을 순회 후<br>
k가 남았다면 뒷수부터 제거하는 방식으로 문제를 풀었다<br>

ex)<br>
4177252841 라는 숫자가 있고 K가 8이상이라면<br>
앞의 숫자들을 최대한 제거하고 가장 큰 수인 '8'까지 도달하는 것이 목표일 것<br>
(반대로 8을 지우지 않은 경우, 최대 숫자를 만들 수 없음)<br>

- 그렇기에 결과 string의 '가장 뒤'와<br>
  현재 비교 문자 c 를 비교함에 따라<br>
  '가장 뒤'에서 pop_back 하는 부분은<br>
  stack 자료구조를 고려할 수 있다<br>
  (다만 stl 컨테이너들 중 string은 이미 pop_back과 push_back이<br>
  존재하기에 별도로 stack을 사용하진 않았음)<br>

- 또한 '현재' 작은 문제들에서 '최선'값을 찾는 것이<br>
  전체적인 최선 값을 찾는 방식이므로<br>
  '그리디 알고리즘'이라 표현할 수 있다<br>

## 제출 코드

```
#include<iostream>
#include<string>

using namespace std;

int main()
{
	int n, k;
	cin >> n >> k;

	string str;
	cin >> str;

	string ret = "";
	ret.push_back(str[0]);

	for (int i = 1; i < n; i++)
	{
		// c를 지워가면서 이전 ret의 back 부분과 비교하기
		char c = str[i];

		while (ret.empty() == false)
		{
			if (k > 0 && c > ret.back())
			{
				k--;
				ret.pop_back();
			}
			else
				break;
		}

		ret.push_back(c);
	}

	while (k > 0)
	{
		k--;
		ret.pop_back();
	}

	cout << ret;
}
```

## 결과
<img width="1155" height="164" alt="Image" src="https://github.com/user-attachments/assets/403a10f7-302b-475e-9f13-86ebe308d288" /><br>

일부 틀린 부분은 마지막 while 문에서<br>
k-- 를 빼뜨려 발생한 문제였다...<br>