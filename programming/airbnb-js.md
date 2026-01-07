---
title: Airbnb JS 스타일 가이드
date: 2025-10-22
tags:
  - JavaScript
  - 스타일가이드
---

여기서 주의해야 할 것들만 좀 추려본다.
기본적으로 *eslint*의 규칙을 따르면서 세부사항이 추가되어 있다. [https://eslint.org/docs/latest/rules/](https://eslint.org/docs/latest/rules/)

# References 참조
- `var`을 지양한다 : `var`는 *function-scoped, 함수 스코프*를 가진다. (`const`와 `let`은 *블록 스코프 block-scoped*)

# Object 객체
- *literal syntax*를 통해 새로운 객체를 만든다. `const student = {name:"name", age:20}`
- `Object.assign` 대신 *전개 구문 spread syntax*를 사용하여 **얕은 복사**를 한다.

# Array 배열
- 객체와 마찬가지로 리터럴로 배열을 생성한다. `new Array()` 사용금지.
- Iterable한 객체를 배열로 변환할 때 전개 구문을 사용한다. 
- `Array.from`은 매핑함수가 있을때 사용한다. `[...foo].map(bar)` 대신 `Array.from(foo,bar)`.
	- 앞의 방법을 사용하면 중간에 필요없는 임시 배열을 만들게 된다.

# Destructuring 구조 분해 할당
- 객체의 여러 속성에 접근하고 사용해야 할 경우에 **구조 분해 할당**을 사용한다.
- 여러 값을 반환할 때 배열 구조 분해 할당 대신, 객체 구조 분해 할당을 사용한다. 
	- `return [one,two,three];`대신 `{one,two,three}`를 반환하면 **반환값의 순서**를 신경쓰지 않아도 되고, 필요 없는 반환값까지 반환받을 필요도 없어진다.

# String 문자열
- **작은 따옴표** `''`를 사용한다.
- 문자열이 **길더라도 한 줄**에 작성한다. `\`이나 `+`를 사용하여 여러 줄에 나눠 작성하지 않는다.
- `eval()`함수를 절대 문자열에 사용하지 않는다.

# Functions 함수
- **함수 선언식** 대신 (자세한 이름을 가지는)*함수 표현식*을 사용한다.
	- `function funcA(){...}` 대신 `const funcA = function longUniqueMoreDescriptiveLexical(){...}
- 기본값이 있는 매개변수는 마지막에 넣는다.

# Classes 클래스
- 클래스 메소드는 메소드 체이닝을 위해 `this`를 반환 해도 된다.
- 클래스 메소드들은 반드시 `this`를 사용해야 한다. [class-methods-use-this](https://eslint.org/docs/latest/rules/class-methods-use-this) 
	- `this`를 사용하지 않는 메소드는 `static`로 만든다.
	- 