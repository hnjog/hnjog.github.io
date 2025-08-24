---
title: "백준 Gold 5 회문"
date : "2025-08-25 09:00:00 +0900"
last_modified_at: "2025-08-25T09:00:00"
categories:
  - 코딩 테스트
tags:
  - 문자열
  - 투 포인터
---

## 회문 (백준 Gold 5)
<https://www.acmicpc.net/problem/17609><br>

회문(팰린드롬)은 '앞 뒤' 방향에서 볼 때<br>
'같은 순서'의 문자로 구성된 문자열을 말한다<br>
(ex - abba,kayak)<br>

회문은 아니지만<br>
만약 임의의 '한 문자'를 삭제하여<br>
회문이 된다면 '유사회문'으로 부르려 한다<br>
(ex - summuus : 5,6번째의 u를 제거하면 회문)<br>

임의의 문자열이 주어졌을때<br>
- 회문이면 0<br>
- 유사회문이면 1<br>
- 전부 아니라면 2<br>
를 반환하는 문제<br>

### 풀이 방식

일단 문자열의 '앞 뒤'를 모두 봐야 한다<br>

그렇기에 앞과 뒤를 조건부로 움직이는 '투 포인터'를<br>
이용하여 풀 수 있는 문제라 생각하였다<br>

- 투 포인터?<br>
  : 배열, 리스트 같은 '선형구조'의 컨테이너에서<br>
    두 개의 '인덱스'(포인터)를 이용하여<br>
	원하는 조건을 만족하는 '구간'이나 '쌍'을 찾는 방식<br>
	-> 일반적인 전체 순회 방식으로 쌍을 찾으면 O(N^2)<br>
	   하지만 투 포인터는 O(N)이나 O(logN)으로 줄일 수 있음<br>
    '정렬된 컨테이너', '연속된 구간', '문자열' 등에서 응용하기 쉽다<br>
	(조건에 따라 '한쪽'만 움직일지, '양쪽'을 다 움직일지 정하는 것이 핵심)<br>

그러면 '조건'을 따지면<br>

- start가 end 값의 이상이라면 종료<br>
  (start == end 이거나 start > end까지 온 순간<br>
  이미 조건을 만족한 셈이다)<br>
  이 때, '한 칸' 넘어갔었다면 1을<br>
  아니면 0을 return

- 같지 않다면<br>
  useCount 여부를 확인<br>
  - 이미 사용했다면 return 2로 실패처리<br>
  - 그렇지 않다면 start 와 end를 각각 한칸 옮긴<br>
    재귀 분귀를 나누게 된다<br>
	(이때도 start 쪽에서 찾았다면 end는 딱히 돌리지 않는다)<br>

## 제출 코드

```
#include<iostream>
#include<string>

using namespace std;

int recur(const string& str,int start, int end, bool useCount)
{
	if (start >= end)
	{
		if (useCount)
			return 1;
		
		return 0;
	}

	if (str[start] == str[end])
	{
		return recur(str, start + 1, end - 1, useCount);
	}

	if (useCount == false)
	{
		int v = recur(str, start + 1, end, true);
		if (v == 2)
			v = recur(str, start, end - 1, true);

		return v;
	}

	return 2;
}


int main()
{
	int t;
	cin >> t;

	string str;
	for (; t > 0; t--)
	{
		cin >> str;
		cout << recur(str, 0, str.size() - 1, false) << '\n';
	}

	return 0;
}
```

## 결과
<img width="1151" height="91" alt="Image" src="https://github.com/user-attachments/assets/b9aa5bb4-b445-4a71-bc07-1774073d1c16" /><br>

문자열 문제이긴 하지만<br>
투 포인터의 개념이 필요한 문제였다<br>
