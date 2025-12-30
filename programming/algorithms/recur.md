---
date: 2025-08-03
tags:
  - 알고리즘
  - 재귀
title: 재귀와 반복이 같이 사용되는 경우
---
# push-재귀-pop-재귀 vs push-재귀-pop
## N-Queen 문제

```js
function solution1(n) {
  const answer = [];
  function isSafe(queens,row,col) {
    for(const comp of queens){
      const r = comp[0];
      const c = comp[1];
      if(row===r||col===c|| Math.abs(row-r)===Math.abs(col-c)){ // Math.abs안써서 틀렸었음
        return false;
      }
    }
    return true;
  }

  function solve(queens,row){
    if(row===n){
      answer.push([...queens]); 
      return;
    }

    for(let i=0;i<n;i++){
      const flag = isSafe(queens,row,i);
      if(flag){
        queens.push([row,i]);
        solve(queens,row+1);
        queens.pop();
        //solve(queens.row+1); //이거 넣었으면 안됐음.
      }
    }

  }

  solve([],0);
  console.log(answer);
  return answer.length;
}

// console.log("N-Queens (4):", solution1(4));
console.log("N-Queens (8):", solution1(8));
```

백트랙킹 문제를 풀다보면 언제는 
재귀함수 후에 `pop()`해서 넣었던걸 뺀 후에 다시 재귀함수를 돌리는 경우가 있고, (push-재귀-pop-재귀)
어쩔때는 push-재귀-pop 까지만 한다.
## 형제 노드 탐색
*순열생성*과 같이 형제 노드들을 모두 탐색해야 할 때는 push-재귀-pop-재귀.
모든 가능한 경우를 탐색해야할 때.
### 그런데 내가 위의 N-Queen 문제도 모든 경우를 탐색하는 문제 아닌가?
위의 코드에서는 재귀호출 안에 for문으로 반복을 보장해줬기 때문에 마지막 재귀함수 호출이 없어도 된다.
```js
function solveRecursive(queens, row, col) {
    if (row === n) {
        answer.push([...queens]);
        return;
    }
    if (col >= n) return; 
    
    if (isSafe(queens, row, col)) {
        queens.push([row, col]);        
        solveRecursive(queens, row+1, 0); 
        queens.pop();                   
    }
    
    solveRecursive(queens, row, col+1); 
}
```
위와 같이 코드를 작성하면 내가 원래 이해하고있던 재귀함수의 형태가 된다.

# 재귀 + 반복이 필요한 유형
## 순열 생성
```js
function solution4(arr) {
  const result = [];
  const visited = new Array(arr.length).fill(false);
  // console.log(visited);

  function permutation(perm) {
    if (perm.length === arr.length) {
      result.push([...perm]);
      return;
    }

    for (let i = 0; i < arr.length; i++) {
      // console.log(arr[i],visited[i]);
      if (!visited[i]) {
        perm.push(arr[i]);
        visited[i] = true;
        permutation(perm);
        perm.pop();
        visited[i] = false;
      }
    }
  }

  permutation([]);
  return result;
}
```
순열생성의 문제에서는 **각 재귀 단계에서 몇 개의 선택지가 있는지 미리 알 수 없기 때문**에 for 반복문을 쓰는것이라고 한다.

