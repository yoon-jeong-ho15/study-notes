---
date: 2025-08-22
tags:
  - substring
  - 스택
  - 프로그래머스
  - 유형인식
title: 문자열 스택
---
# 초안
```js
function solution(s) {
	while (s.length > 0) {
		for (let i = 0; i < s.length; i++) {
			const ch1 = s[i];
			const ch2 = s[i + 1];
			// console.log(ch1, ch2);
			if (ch1 === ch2) {
				s = s.substring(0, i) + s.substring(i + 2);
				// console.log("s : ", s);
				break;
			}
			if (i === s.length - 1) {
				return 0;
			}
		}
	}
	return 1;
}

// console.log(solution("baabaa"));
console.log(solution("cdcd"));
```
`s`문자열 전체를 순회하면서
반복적으로 앞의 두 글자를 비교하고 
전부 다 제거해서 while 문을 빠져나오면 1을 반환하고,
반환하지 못해서 i가 `s.length-1` 즉 문자열 끝까지 도달하면 0을 반환하는 방식으로 풀었다.

정확도는 18/18인데 효율성이 1/8로 심히 낮았다.

## 원인 파악
while과 for 로 반복 중첩이 있어서 $O(n^2)$복잡도를 가진다.
알고리즘의 시간복잡도 에서 $O(n^2)$보다 낮은 복잡도들은
$O(nlogn), O(n), O(logn)$들이 있다. (내림차순)

<image src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FcrD9hM%2FbtrZYSH12da%2FAAAAAAAAAAAAAAAAAAAAAFITBKKsIc7wiSKrqakU1926rn0rg-x4YHoxhOzsgMlJ%2Fimg.webp%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1756652399%26allow_ip%3D%26allow_referer%3D%26signature%3DR5k0UpB3MewaD9lZVx0C1dme2q4%253D" />

저 빨간색 영역이 horrible 하다는걸 보니
프로그래밍을 할 때 최대한 **반복문 중첩**은 피해야되는것 같다.

그럼 위의 문제도 반복문 **중첩 없이 풀어야**된다는게 문제의 출제의도일 것인데, 어떻게 풀어야될까?
일단 안의 for문이 이 함수의 핵심 기능을 담당하고 있기 때문에 밖의 while문의 역할을 대체할 코드를 for 문 안에 작성해야되겠지?

...(3분간 고민후 클로드에게 물어봤다.)

스택을 사용하면 된다고 한다. 이걸 생각 못했다니.
KOCW 자료구조 수업에서도 들었던 내용인데, 스택은 마지막것, 최신것과 비교할 때 사용된다.

## 왜 스택을 생각해내지 못했을까?
코딩테스트에서 스택을 사용해 푼 문제가 몇 개 있기도하고, 스택을 모르던것도 아닌데 왜 이 문제에서 **떠올려내지 못했을까?**
심지어 예전에 풀었던 괄호 짝 맞추기 문제랑 사실상 구조는 똑같은데 말이다....

코딩테스트에서 어떤 알고리즘, 자료유형을 사용해야하는지 파악할 수 있는게 중요하다고 전에 들었던 특강(서울고용센터 6월 9일 특강)이 생각났다.

문제를 *추상화* 할 수 있어야 한다.
여기서 바로 수학실력이 프로그래밍에 개입하는거 아닐까.
`"baabaa"` `"({}){}"` 모두 인접한 쌍을 제거하는게 본질이니까

스택문제의 핵심 *키워드*들로는 다음과 같은 것들이 있다고 한다.
인접한, 연속된, 쌍, 최근, 되돌리기, ...

문제가 요구하는 과제를 본질만 추려서 추상화 한 다음에 해당하는 키워드를 찾는 연습이 필요할 것 같다.

# 답안
```js
function solution(s) {
	const stack = [];

	for (let i = 0; i < s.length; i++) {
		const ch = s[i];
		// console.log(ch);
		if (stack.length > 0 && stack[stack.length - 1] === ch) {
			stack.pop();
		} else {
			stack.push(ch);
		}
	}

	return stack.length > 0 ? 0 : 1;
}
```
이렇게 스택을 사용해 풀었고, 통과했다.
