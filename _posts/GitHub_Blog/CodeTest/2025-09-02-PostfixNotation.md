---
title: "백준 Gold 2 후위 표기식"
date : "2025-09-02 09:00:00 +0900"
last_modified_at: "2025-09-02T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 스택
---

## 후위 표기식 (백준 Gold 2)
<https://www.acmicpc.net/problem/1918><br>

연산자가 피연산자 뒤에 위치하는 표기법을 '후위 표기법'이라 함<br>
ex) a + b -> ab+<br>

중위 표기식이 주어졌을때<br>
그것을 후위 표기식으로 바꾸는 문제<br>

## 풀이 방법

예시에서 주어진 대로<br>
'괄호'를 생각하면 이해하기 쉽다<br>

ex : a+b*c는 (a+(b * c))의 식과 같게 된다<br>
가장 안쪽의 괄호 *의 우선순위가 높으므로<br>
해당 연산자를 자신의 괄호 오른쪽으로 빼<br>
(a + bc*) 가 되며<br>
이후 +를 오른쪽 괄호의 밖으로 빼준다<br>
결과 : abc*+<br>

풀이 팁<br>

- 연산자 우선순위를 고려해야 한다<br>
  ()의 우선순위가 가장 높고<br>
  */가 그 다음<br>
  +-의 순서가 마지막이다<br>

- 다만 '후위'이기에<br>
  특정한 연산자가 '나온 시점'에<br>
  이전 연산자들을 '어떻게 처리'해야 할지 생각<br>

- 스택을 이용해야 함<br>
  '이전 연산자'를 나중에 더하기에<br>
  해당 연산자를 보관할 자료구조가 필요<br>

## 제출 코드

```
#include<iostream>
#include<string>
#include<stack>

using namespace std;

int main()
{
	string s;
	cin >> s;

	stack<char> st;
	string ret = "";

	for (char c : s)
	{
		if (c >= 'A' && c <= 'Z')
			ret.push_back(c);

		if (c == '(')
		{
			st.push(c);
		}

		if (c == '+' ||
			c == '-')
		{
			if(st.empty() == false)
			{
				while (st.empty() == false)
				{
					char temp = st.top();
					if (temp == '+' ||
						temp == '-' ||
						temp == '*' ||
						temp == '/')
					{
						ret.push_back(temp);
						st.pop();
					}
					else
					{
						break;
					}
				}
			}
			st.push(c);
		}

		if (c == '*' ||
			c == '/')
		{
			if (st.empty() == false)
			{
				while (st.empty() == false)
				{
					char temp = st.top();
					if (temp == '*' ||
						temp == '/')
					{
						ret.push_back(temp);
						st.pop();
					}
					else
					{
						break;
					}
				}
			}
			st.push(c);
		}
		
		if (c == ')')
		{
			while (st.empty() == false)
			{
				char temp = st.top();
				st.pop();
				if (temp == '(') 
				{
					break;
				}
				else
				{
					ret.push_back(temp);
				}
			}
		}
	}

	while (st.empty() == false)
	{
		ret.push_back(st.top());
		st.pop();
	}

	cout << ret;

	return 0;
}
```

## 구현

연산자들을 저장할 스택을 준비<br>

- ')' 를 만나는 경우<br>
  이전에 저장한 '('를 만날때까지<br>
  빼주면서 연산자를 결과 문자열에 더해주었다<br>

- */ 를 만나는 경우, 다른 */를 만날때까지<br>
  연산자를 빼주면서 문자열에 더해준다<br>

- +- 는 우선순위가 가장 낮기에<br>
  나온 시점에서 다른 연산자들을<br>
  빼주고 문자열에 더해준다<br>

요점은<br>
'현재 연산자'와 '마지막 연산자'의<br>
우선순위 비교였다<br>

## 결과
<img width="1172" height="133" alt="Image" src="https://github.com/user-attachments/assets/37ddc763-24cd-49c5-8b03-73cf9afddcce" /><br>

처음에는 문제의 '연산자 우선순위'를 고려하지 못한 코드였다<br>
여러모로 스택을 고려한 구현 문제<br>