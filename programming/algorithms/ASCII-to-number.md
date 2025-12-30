---
date: 2025-08-19
tags:
  - 문자열
  - ASCII
  - 모듈러
  - 프로그래머스
title: ASCII -> 문자
---
https://school.programmers.co.kr/learn/courses/30/lessons/155652

`s , skip, index` 세 인자를 받는다.
s는 암호문, s의 각 글자들의 ASCII코드에서 index만큼 더한 문자가 복호화된 문자다.
그런데 `skip`에 해당하는 문자는 index만큼 더할때 제외된다.
a(97) + 5 = f(102) 지만 `skip`안에 b와 d가 있으니 2번은 건너뛰고 5개를 더하니 실상 +7이 되고 a->h가 된다.

자바스크립트의 `chatCodeAt()` 과 `fromCharCode()` 함수를 사용해서 풀었는데,
검색해보니 영어 소문자는 97~122범위에 있었다.
```js
function solution(s, skip, index) {
    var answer = '';
    for(let ch of s){
    	let n = ch.charCodeAt();
    	// console.log(n);
    	let i = 1;
    	let j = 0;

    	while (true){
    		let m = n+i>122? (n+i)%123+97:n+i;
    		let a = String.fromCharCode(m);
    		// console.log(a);
    		if(!skip.includes(a)){
    			j++;
    			// console.log(`${i}, ${j}, !skip.includes(${a})`);
    		}
    		if(j===index){
    			answer += a;
    			break;
    		}
    		i++;
    	}
    }
    return answer;
}
```
이렇게 풀었더니 16/20 개 통과. 
잘못될 부분은 모듈러 연산을 통해 `skip`에 있는 문자인지 확인하는 과정에 있을 수 밖에 없어서 다시 봤다.

틀린 테스트케이스는 아마 index가 커서 `%123` 후에도 이미 알파벳 소문자 범위 26을 넘은 경우일것.
`(n+i)%123`이 26 이상일 경우 여기에 97을 더하면 그 결과는 z(122)를 넘어 알파벳 소문자가 아닌 다른 문자가 나온다.
그래서 `let m = n+i>122? ((n+i-97)%26)+97:n+i;` 이렇게 수정했고 20/20 통과.
