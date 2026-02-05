---
title: 다익스트라에서 우선순위 큐
date: 2025-09-18
tags:
  - 알고리즘
  - 우선순위큐
  - 다익스트라
  - 백준
---
## 1753번 문제
### 초안
```js
const source = `5 6
1
5 1 1
1 2 2
1 3 3
2 3 4
2 4 5
3 4 6`;

const input = source.split("\n");
const [V, E] = input[0].split(" ").map(Number);
const K = Number(input[1]);
// console.log(V, E, K);

let adj = Array(V + 1)
	.fill()
	.map(() => []);

for (let i = 2; i < input.length; i++) {
	const [u, v, w] = input[i].split(" ").map(Number);
	adj[u].push([v, w]);
}
// console.log(adj);

const dist = Array(V + 1).fill(Number.MAX_SAFE_INTEGER);
// console.log(dist);

const queue = [[K, 0]];
// console.log(queue);
dist[K] = 0;

while (queue.length > 0) {
	// console.log(queue);
	const [u, d] = queue.shift();
	// console.log(u, d);
	for (const [v, w] of adj[u]) {
		//[2,2] [3,3]
		if (dist[v] > d + w) {
			dist[v] = d + w;
			queue.push([v, dist[v]]);
		}
	}
}

// console.log(dist);
for (let i = 1; i < dist.length; i++) {
	console.log(dist[i] < Number.MAX_SAFE_INTEGER ? dist[i] : "INF");
}

```

이렇게 제출했더니 **시간초과**가 발생했다. 
그래서 다음과 같은 코드를 추가했다 :
- `visited` 배열을 통해 **방문한 노드 관리** `if(visited[u]) continue;`
- `while`반복문 시작할때 마다 `queue.sort((a,b)=>a[1]-b[1]);`로 큐를 정렬해 **우선순위 큐**를 구현

그래도 여전히 **메모리 초과**라고 한다.
자바스크립트의 `sort()` 함수는 정렬용 임시 공간을 필요로 하기 때문인듯 한데, 그래서 힙정렬을 구현해서 제출했더니 통과했다.

### 답안
```js
while (queue.length > 0) {
	// console.log(queue);
	// queue.sort((a, b) => a[1] - b[1]);
	const [u, d] = extract();
	// console.log(u, d);

	if (d > dist[u]) continue;

	for (const [v, w] of adj[u]) {
		//[2,2] [3,3]
		if (dist[v] > d + w) {
			dist[v] = d + w;
			insert([v, dist[v]]);
		}
	}
}

// console.log(dist);
for (let i = 1; i < dist.length; i++) {
	console.log(dist[i] < Number.MAX_SAFE_INTEGER ? dist[i] : "INF");
}

function insert(node) {
	queue.push(node);
	let i = queue.length - 1;
	while (i > 0) {
		const parent = Math.floor((i - 1) / 2);
		if (queue[parent][1] <= queue[i][1]) break;
		[queue[parent], queue[i]] = [queue[i], queue[parent]];
		i = parent;
	}
}

function extract() {
	if (queue.length === 0) return null;
	if (queue.length === 1) return queue.pop();

	const min = queue[0];
	queue[0] = queue.pop();

	let i = 0;
	while (true) {
		const left = 2 * i + 1;
		const right = 2 * i + 2;
		let smallest = i;

		if (left < queue.length && queue[left][1] < queue[smallest][1]) {
			smallest = left;
		}
		if (right < queue.length && queue[right][1] < queue[smallest][1]) {
			smallest = right;
		}

		if (smallest === i) break;
		[queue[i], queue[smallest]] = [queue[smallest], queue[i]];
		i = smallest;
	}

	return min;
}
```

