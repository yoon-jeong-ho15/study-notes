---
date: 2025-08-21
tags:
  - 알고리즘
  - 프로그래머스
  - 정렬
  - TimSort
  - 클로드
title: 최솟값 만들기
---

[문제](https://school.programmers.co.kr/learn/courses/30/lessons/12941)

# 초안 (삽입정렬)

```js
function solution(A, B) {
  var answer = 0;
  const arr1 = [];
  const arr2 = [];

  for (const a of A) {
    // console.log(a);
    let inserted = false;
    for (let i = 0; i < arr1.length; i++) {
      if (a < arr1[i]) {
        arr1.splice(i, 0, a);
        inserted = true;
        break;
      }
    }
    if (!inserted) arr1.push(a);
  }

  for (const b of B) {
    // console.log(b);
    let inserted = false;
    for (let i = 0; i < arr2.length; i++) {
      if (b < arr2[i]) {
        arr2.splice(i, 0, b);
        inserted = true;
        break;
      }
    }
    if (!inserted) arr2.push(b);
  }

  console.log(arr1, arr2);

  for (let i = 0; i < arr1.length; i++) {
    answer += arr1[i] * arr2[arr2.length - 1 - i];
  }

  return answer;
}

console.log(solution([1, 4, 2], [5, 4, 4]));
```

왜 이렇게 쉽나 했다.
처음에 이렇게 풀었더니 효율성 테스트에서 0/4점.
정렬이 방법이 문제인거같은데 일반정렬 말고 내장 `sort()`를 쓰면 해결될 게 분명한데,

# 수정안(병합 정렬)

그러면 너무 쉬우니까 병합정렬로 풀어보려고 한다.

```js
function solution(A, B) {
  var answer = 0;
  const arr1 = mergeSort(A);
  const arr2 = mergeSort(B);

  function mergeSort(arr) {
    if (arr.length == 1) {
      return arr;
    }

    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));

    const temp = [];

    let i = 0;
    let j = 0;
    while (i < left.length && j < right.length) {
      const n = left[i];
      const m = right[j];
      if (n > m) {
        temp.push(n);
        i++;
      }
      if (m >= n) {
        temp.push(m);
        j++;
      }
    }

    while (i < left.length) {
      temp.push(left[i]);
      i++;
    }
    while (j < right.length) {
      temp.push(right[j]);
      j++;
    }

    return temp;
  }

  console.log(arr1, arr2);

  for (let i = 0; i < arr1.length; i++) {
    answer += arr1[i] * arr2[arr2.length - 1 - i];
  }

  return answer;
}
```

그런데 또 실패.

`sort()` 내장 함수로 정렬하니까 통과했다..

# Tim sort

참고 :
https://velog.io/@gusdh2/sort
https://d2.naver.com/helloworld/0315536

자바스크립트 내장 함수 `sort()`는
각 브라우저가 사용하는 js엔진마다 다르게 구현됐다고 한다.
그런데 가장 많이 사용되는 V8의 경우(node.js가 사용)는 Tim Sort알고리즘을 사용한다고 한다.

Tim sort는 삽입 정렬과 병합 정렬을 결합하여 사용한다고 하는데,
최선은 n, 평균 n log n, 최악 n log n 의 복잡도를 가진다고 한다.

<image src="https://d2.naver.com/content/images/2020/01/img.png" />

위의 표를 보면 병합 정렬과 평균, 최악은 똑같은데 최선에서 Tim sort가 더 빠르다.
그런데 최선의 경우에는 삽입정렬과 같은데,
왜 내가 구현했을때는 둘 다 실패했을까?

클로드에게 물어봤다.

> 
> # `sort()` vs 직접구현
> 
> 정말 좋은 분석입니다! Tim Sort가 성공한 이유는 **시간복잡도가 같아도 실제 성능은 완전히 다르기 때문**입니다.
> 
> ## 1. 네이티브 코드 vs JavaScript 코드
> 
> ```javascript
> // JavaScript로 직접 구현 - 느림
> for (let i = 0; i < arr1.length; i++) {
>   if (a < arr1[i]) {
>     arr1.splice(i, 0, a); // 인터프리터에서 실행
>     break;
>   }
> }
> 
> // 내장 sort() - 빠름
> arr.sort((a, b) => a - b); // C++로 구현된 네이티브 코드 실행
> ```
> 
> **차이점:**
> 
> - **직접 구현**: JavaScript 엔진이 한 줄씩 해석하며 실행
> - **Tim Sort**: V8 엔진 내부의 **C++로 컴파일된 코드**로 실행
> 
> ## 2. 메모리 접근 패턴의 최적화
> 
> ### 직접 구현한 삽입정렬의 문제:
> 
> ```javascript
> arr1.splice(i, 0, a); // 배열 중간에 삽입
> ```
> 
> - **메모리 이동**: 삽입 위치 뒤의 모든 요소를 한 칸씩 밀어야 함
> - **메모리 재할당**: 배열 크기 변경으로 인한 메모리 복사
> - **캐시 미스**: 불규칙한 메모리 접근 패턴
> 
> ### Tim Sort의 최적화:
> 
> - **제자리 정렬**: 불필요한 메모리 할당 최소화
> - **캐시 친화적**: CPU 캐시를 효율적으로 활용
> - **SIMD 명령어**: 병렬 처리 명령어 활용 가능
> 
> ## 3. 알고리즘 레벨의 최적화
> 
> Tim Sort는 단순한 병합정렬이 아니라:
> 
> ### 적응적 정렬:
> 
> ```
> 거의 정렬된 데이터: O(n)
> 랜덤 데이터: O(n log n)
> 역순 데이터: O(n) (런을 역순으로 처리)
> ```
> 
> ### 런(Run) 감지:
> 
> - 이미 정렬된 구간을 미리 찾아서 활용
> - 불필요한 비교 연산 대폭 감소
> 
> ### 갤로핑 모드:
> 
> - 한쪽 배열의 요소가 연속으로 선택될 때 이진 탐색으로 가속화
> 
> ## 4. 실제 성능 비교 예시
> 
> ## 5. 코딩테스트에서의 실제 차이
> 
> ### 시간 제한 예시 (1초):
> 
> - **Tim Sort**: 10만 개 요소도 0.01초 이내
> - **직접 구현 삽입정렬**: 1000개 요소도 1초 초과
> - **직접 구현 병합정렬**: 10만 개 요소에 0.5초 (아슬아슬)
> 
> ## 결론
> 
> 같은 시간복잡도여도 **실제 성능은 완전히 다릅니다**:
> 
> 1. **구현 언어**: C++ vs JavaScript
> 2. **메모리 효율성**: 제자리 정렬 vs 추가 메모리 할당
> 3. **하드웨어 최적화**: CPU 캐시, SIMD 명령어 활용
> 4. **알고리즘 최적화**: 적응적 정렬, 런 감지 등
> 
> **교훈**: 이론적 시간복잡도와 실제 성능은 별개입니다. 코딩테스트에서는 **검증된 내장 함수를 활용**하는 것이 현명한 선택이에요!
