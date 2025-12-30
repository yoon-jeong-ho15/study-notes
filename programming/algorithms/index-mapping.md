---
title: n^2배열 자르기 문제
index:
date: 2025-09-04
tags:
  - 시간초과
  - 인덱스매핑
  - 수학적최적화
  - 범위제한
---
# n^2 배열 자르기 문제
https://school.programmers.co.kr/learn/courses/30/lessons/87390
## 초안
```js
function solution(n, left, right) {
    var answer = [];
    for (let i = 1; i <= n; i++) {
        // console.log("i :", i);
        for (let j = 1; j <= n; j++) {
            const k = j > i ? j : i;
            answer.push(k);
        }
    }
    // console.log(answer);
    answer = answer.slice(left, right + 1);

    return answer;
}
```
처음엔 이렇게 풀었더니 *런타임 에러* 발생.
메모리 공간을 많이 차지하면 런타임 에러가 발생한다고 하는데, 그거 때문인것 같다.
문제 조건에서 주어지는 n의 범위를 `1 <= n <= 10^7`이라고 하니까 최대 10^14 개의 숫자를 담는 배열이 필요하게 된다.

# 수정안
```js
function solution(n, left, right) {
	var answer = [];
	let count = 0;
	for (let i = 0; i < n; i++) {
		for (let j = 0; j < n; j++) {
			const k = (j > i ? j : i) + 1;
			if (count >= left && count <= right) {
				answer.push(k);
			}
			if (count > right) {
				break;
			}
			count++;
		}
		if (count > right) {
			break;
		}
	}

	return answer;
}
```

이렇게 풀었더니 8/20개가 시간초과를 실패했다.
반복문 중첩으로 인한 $O(n^2)$의 복잡도가 문제인거 같다. 
아무리 `count>right`이면 빠져나온다고 해도 right가 워낙 크면 문제가 될 수도.

## 답안
```js
function solution(n, left, right) {
	const answer = [];

	for (let a = left; a <= right; a++) {
		const i = Math.floor(a / n);
		const j = a % n;
		// console.log(j, k);

		const k = (j > i ? j : i) + 1;
		answer.push(k);
	}

	return answer;
}
```

핵심은 초안과 수정안에서는 불필요한 연산까지 한 다음에 조건을 확인해서 `answer` 배열에 추가 했던것.
문제에는 조건이 이렇게 있다.
1. left이상 right 이하 순서의 숫자만 담을것.
2. 넣을 숫자는 i와 j중에 큰 것(에 +1)으로 할것.

여기서 연산 횟수를 줄일 수 있는 앞의 조건을 간과했다.
앞으로 이렇게 *범위 제한*이 있는 문제는 시간복잡도를 신경써야 한다는 문제로 봐야겠다.
