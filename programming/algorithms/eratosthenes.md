---
date: 2025-08-12
tags:
  - 수학
  - 소수
  - 프로그래머스
title: 에라토스테네스의 체
---
```js
/ 시간초과 7
function solution2(n) {
    var answer = 0;
    let arr = [];
    for(let i = 2;i<=n;i++){
    	let flag = true;
    	for(let j = 2;j<i;j++){
    		if(i%j===0){
    			flag = false;
    			break;
    		}
    	}
    	if(flag) arr.push(i);
    }
    // console.log(arr);
    return arr.length;
}

// 시간초과 5개
function solution3(n){
	let arr = [];
	for(let i = 2;i<=n;i++){
    	let flag = true;
    	for(const pn of arr){
    		if(i%pn===0){
    			flag = false;
    			break;
    		}
    	}
    	if(flag) arr.push(i);
    }
    // console.log(arr);
    return arr.length;
}

// 에라토스테네스의 체
function solution(n) {
    let isPrime = new Array(n + 1).fill(true);
    isPrime[0] = isPrime[1] = false;
    
    for(let i = 2; i * i <= n; i++) {
        if(isPrime[i]) {
            for(let j = i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }
   	let arr = [];
   	for (let i=2;i<isPrime.length;i++){
   		if(isPrime[i]){
   			arr.push(i);
   		}
   	}
   	// console.log(arr);
    return arr.length;
}

// console.log(solution(1000));
// console.log(solution(5));
console.log(solution(1000000));
```

맨 처음 작성한 solution2는 $O(n^2)$의 복잡도를 가진다.
n개의 입력마다 i반복마다 j반복을 돌리니까.

두 번째로 작성한 solution3은?
1) 일단 i반복이 하나 있으니까 n,
2) i 반복 안에서 arr의 크기만큼(현재까지 구한 소수의 수) 반복을 한다. (매 번 달라진다.)
arr의 크기는 대략 $n/log{n}$개라고 한다. (n=10, 10/log10= 4고 실재로 2,3,5,7로 4개가 있다.
그래서 $O(n^2/logn)$ 복잡도를 가진다. n번 반복마다 n/logn개의 숫자와 확인하니까.

마지막으로 작성한 solution은 '에라토스테네스의 체' 방법이다.
1) 미리 n까지의 인덱스를 가지는 배열을 만들고, 다 true값을 넣어준다. (n+1개가 된다)
2) i반복을 통해 위의 배열을 순회하면서 값이 true인 경우 그 index의 모든 배수의 index를 다 false로 바꿔준다.

그런데 단순히 1번과 2번을 곱하는 방식이 아니라 조금 더 복잡하게 계산해야한다.
$\sum{n/p}$이고 이는 $log{log{n}}$에 수렴한다고. 그래서 $O(log{log{n}})$복잡도를 가진다.








