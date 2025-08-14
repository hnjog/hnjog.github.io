---
title: "백준 Silver 2 쇠막대기"
last_modified_at: "2025-08-14T10:00:00"
categories:
  - 코딩 테스트
tags:
  - 스택
---

## 쇠막대기 (백준 Silver 2)
<https://www.acmicpc.net/problem/10799><br>

스택을 사용하여 푸는 문제이다<br>

### 스택을 사용하는 이유?
'가장 마지막에 들어온' 막대를 닫기에<br>
스택을 사용하는 것이 직관적이다<br>
(LIFO)<br>

이번에는 단순한 개수만 구하는 것이기에<br>
사실 int 등의 변수 하나로 대체가 가능하지만<br>
별도의 데이터 처리가 있을 경우 등을 고려하면<br>
스택을 사용하는 것이 정석<br>

### 실제 푸는 방법

<img width="2147" height="1217" alt="Image" src="https://github.com/user-attachments/assets/bfc72a9d-cc1f-4929-a7ef-2767154362d0" /><br>

요점은 '레이저()'를 구분하는 것이다<br>
따라서 '(' 이후 바로 ')'가 들어온다면<br>
그것은 레이저 이며<br>

'잘린 막대'는 stack에 여태까지 쌓인<br>
막대들( '(' ) 이므로<br>
stack.size()를 정답에 더해준다<br>

또한 ')'를 만난 경우<br>
이제 마지막 남은 '꽁다리' 막대 하나를<br>
정답에 더해준다<br>


## 결과

<img width="1173" height="106" alt="Image" src="https://github.com/user-attachments/assets/301dd216-fb73-4609-b2ff-ca9b47c212d4" /><br>

### 제출 코드

```
#include<iostream>
#include<stack>
#include<string>

using namespace std;

int main()
{
	stack<char> st;
	string str;
	cin >> str;

	size_t idx = 0;
	size_t ssize = str.size();

	int answer = 0;

	while (idx < ssize)
	{
		if (str[idx] == '(')
		{
			if (str[idx + 1] == ')')
			{
				answer += st.size();
				idx++;
			}
			else
			{
				st.push('(');
			}
		}
		else
		{
			if (st.empty() == false)
			{
				answer++;
				st.pop();
			}
		}
		idx++;
	}

	cout << answer;

	return 0;
}
```