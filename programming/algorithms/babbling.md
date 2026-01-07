---
date: 2025-08-18
tags:
  - 문자열
  - 알고리즘
  - 프로그래머스
title: 옹알이
link:
  - https://school.programmers.co.kr/learn/courses/30/lessons/133499
---

```js
function solution2(babbling) {
    var answer = 0;
    const arr = ["aya","ye","woo","ma"];
    const result = [];
    for(let i=0;i<babbling.length;i++){
    	let str = babbling[i];
    	// console.log(str);
    	for(const word of arr){
	    	if(str.includes(word)){
	    		// console.log(`${str} includes ${word}`)
	    		const bindex = str.indexOf(word);
	    		const eindex = str.indexOf(word)+word.length;
	    		str = str.substring(0,bindex)+str.substring(eindex);
	    	}
    	}
    	if(str===""){
  			result.push(babbling[i]);
    	}
    }
    // console.log(result);
    return result.length;
}
```
이렇게 했더니 정확도가 55/100

이해가 안돼서 클로드한테 물어봤더니 자기도 모르겠다며 어떤 경우에 틀렸는지 테스트케이스를 알려달라고 한다. 그걸 알았다면 처음부터 말해줬겠지..

그래서 너라면 어떻게 풀거같은지 코드를 작성해달라고 해서 그걸 제출해보니 100점 통과.
```js
function solution(babbling) {
    const validSounds = ["aya", "ye", "woo", "ma"];
    let count = 0;
    
    for (const word of babbling) {
        if (canPronounce(word, validSounds)) {
            count++;
        }
    }
    
    return count;
}

function canPronounce(word, validSounds) {
    let i = 0;
    let prevSound = "";
    
    while (i < word.length) {
        let found = false;
        
        // 현재 위치에서 시작하는 유효한 발음 찾기
        for (const sound of validSounds) {
            if (word.startsWith(sound, i)) {
                // 연속 발음 체크
                if (sound === prevSound) {
                    return false;
                }
                
                prevSound = sound;
                i += sound.length;
                found = true;
                break;
            }
        }
        
        // 어떤 유효한 발음도 찾지 못했다면 발음 불가능
        if (!found) {
            return false;
        }
    }
    
    return true;
}
```

# 비교
```js
function solution2(babbling) {
    var answer = 0;
    const arr = ["aya","ye","woo","ma"];
    const result = [];
    for(let i=0;i<babbling.length;i++){
    	let str = babbling[i];
    	// console.log(str);
    	for(const word of arr){
	    	if(str.includes(word)){
	    		// console.log(`${str} includes ${word}`)
	    		const bindex = str.indexOf(word);
	    		const eindex = str.indexOf(word)+word.length;
	    		str = str.substring(0,bindex)+str.substring(eindex);
	    	}
    	}
    	if(str===""){
  			result.push(babbling[i]);
    	}
    }
    // console.log(result);
    return result.length;
}
```
내 방식은 발음할 수 있는 단어들의 배열 `arr`을 만들고,
babbling에 있는 단어들 하나씩 `arr`의 요소들을 가지고있는지 확인하고 있으면 뺀다.

```js
function solution(babbling) {
    const validSounds = ["aya", "ye", "woo", "ma"];
    let count = 0;
    
    for (const word of babbling) {
        if (canPronounce(word, validSounds)) {
            count++;
        }
    }
    
    return count;
}

function canPronounce(word, validSounds) {
    let i = 0;
    let prevSound = "";
    
    while (i < word.length) {
        let found = false;
        
        // 현재 위치에서 시작하는 유효한 발음 찾기
        for (const sound of validSounds) {
            if (word.startsWith(sound, i)) {
                // 연속 발음 체크
                if (sound === prevSound) {
                    return false;
                }
                
                prevSound = sound;
                i += sound.length;
                found = true;
                break;
            }
        }
        
        // 어떤 유효한 발음도 찾지 못했다면 발음 불가능
        if (!found) {
            return false;
        }
    }
    
    return true;
}
```
클로드의 방식은 발음 가능한 단어들인 `validSounds`에서 모든 요소들을 `babbling`의 요소 하나씩 검증하는건 나와 같다.
그런데 `while (i < word.length)`조건 아래 `validSounds`의 요소들을 계속 반복한다는 차이가 있다.
그리고 `includes()`가 아니라 `startsWith()`로 비교하면서 순차적으로 `i`를 늘려나간다. 

# 가설
위에 비교를 적다보니까 자연스럽게 떠올랐다.
혹시 중복사용되지만 반복되지는 않는 단어가 문제일까?
문제에서는 '같은 발음의 연속'을 할 수 없다고 했지 중복해서 사용할수 없다고는 한 적 없으니까.
가령 "yemaye"는 내 solution2로는 "yemaye"에서 "ye"와 "ma"를 한번씩 밖에 제거해주지 못하기 때문에 "ye"가 결과적으로 남게된다. 

그래서 내가 처음에 작성한 solution2를 다음과 같이 여러번 제거할 수 있도록 수정했다.
```js
function solution2(babbling) {
    var answer = 0;
    const validSounds = ["aya","ye","woo","ma"];

    const result = [];
    for(let i=0;i<babbling.length;i++){
    	let word = babbling[i];
    	// console.log(word);
    	let okay = true;
    	let prevSound = "";
    	while(word.length>0){
    		let flag = false;
    		for(const sound of validSounds){
    			if(word.startsWith(sound)){
    				if(sound === prevSound){
    					break;
    				}
    				word = word.substring(sound.length);
    				prevSound = sound;
    				flag = true;
    			}
    		}
    		if(!flag){
    			okay = false;
    			break;
    		}
    	}
    	if(okay){
    		result.push(babbling[i]);
    	}

    }
    // console.log(result);
    return result.length;
}
```

매 `word`별로 `word.length>0` 인 동안 계속해서 `validSounds`의 요소들로 시작하는지 확인해서 그렇다면 `sound`의 길이만큼 word의 앞부분을 잘라줬다.
그리고 클로드 답안처럼 `prevSound = sound`를 해줘서 **같은 발음이 연속되지 않게** 처리했다.
`flag`는 `validSounds`를 한바퀴 돌면서 확인했을때 
1) 아무 `sound`로 시작하면서 
2) 2)`sound===prevSound`가 아닌 경우에 참이된다. 
`validSounds`를 한바퀴 순회했는데 `flag`가 false라면 바로 `okay = false`로 하고 해당 `word`는 건너뛰고 다음 `word`를 검증한다.
`word.length > 0` 조건이 있으니까 성공적으로 발음할 수 있는 `word`의 경우 word가 빈 문자열 `""`이 될 때 까지 `flag===true` 상태이고 그러니 `okay===ture`상태로 남고 `result`에 추가된다. 

