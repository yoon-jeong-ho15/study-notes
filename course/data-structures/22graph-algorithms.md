---
title: 그래프 알고리즘
date: '2025-09-13'
order: 22
---
# 최단경로 알고리즘
## Unweighted graph (BFS)
*너비 우선 탐색 BFS* 알고리즘을 사용한다.
가중치가 없는 그래프에서 시작점에서 목표점까지의 최단 경로는 **가장 적은 수의 간선**을 가지는 경로를 의미한다.
BFS알고리즘은 복잡하게 얽혀있는 그래프를 **시작점을 루트로 하는 트리 구조로 재구성**을 하는것과 비슷하다(고 생각한다). 
그래서 목적지 노드가 속한 레벨을 구하면 그게 곧 최단 거리가 되고 해당 경로 상의 간선들의 모음이 최단경로가 된다.

사건 복잡도는 O(V+E). 

```js
function bfsShortestPath(graph, start, end) {
    const queue = [[start, [start]]];
    const visited = new Set([start]);
    
    while (queue.length > 0) {
        const [node, path] = queue.shift();
        
        if (node === end) {
            return path;
        }
        
        for (const neighbor of graph[node]) {
            if (!visited.has(neighbor)) {
                visited.add(neighbor);
                queue.push([neighbor, [...path, neighbor]]);
            }
        }
    }
    
    return null;
}

const graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'D'],
    'D': ['B', 'C', 'F'],
    'E': ['B', 'F'],
    'F': ['D', 'E']
};

console.log(bfsShortestPath(graph, 'A', 'F'));
```

## Weighted graph (다익스트라)
Dijkstra에 의해 고안된 알고리즘.
다익스트라 알고리즘은 BFS알고리즘을 **일반화**한 것이다. (왜 '일반화'라고 하냐면, unweighted graph는 모든 간선의 가중치가 1로 동일한 매우 특수한 weighted graph이기 때문에.)

```js

class MinHeap {
    constructor() {
        this.heap = [];
    }

    push(val) {
        this.heap.push(val);
        this._heapifyUp(this.heap.length - 1);
    }

    pop() {
        if (this.size() === 0) return null;
        if (this.size() === 1) return this.heap.pop();

        const min = this.heap[0];
        this.heap[0] = this.heap.pop();
        this._heapifyDown(0);
        return min;
    }

    size() {
        return this.heap.length;
    }

    _heapifyUp(index) {
        while (index > 0) {
            const parent = Math.floor((index - 1) / 2);
            if (this.heap[parent][0] <= this.heap[index][0]) break;
            [this.heap[parent], this.heap[index]] = [this.heap[index], 
                                this.heap[parent]];
            index = parent;
        }
    }

    _heapifyDown(index) {
        const n = this.heap.length;
        while (true) {
            let smallest = index;
            const left = 2 * index + 1;
            const right = 2 * index + 2;

            if (left < n && this.heap[left][0] < this.heap[smallest][0]){
                smallest = left;
            }
            
            if (right < n && this.heap[right][0] < this.heap[smallest][0]){
                smallest = right;
            }

            if (smallest === index) break;
            [this.heap[smallest], this.heap[index]] = 
                              [this.heap[index], this.heap[smallest]];
            index = smallest;
        }
    }
}

// Function to construct adjacency 
function constructAdj(edges, V) {
    // adj[u] = list of [v, wt]
    const adj = Array.from({ length: V }, () => []);
    // const adj = Array(V).fill().map(()=>[]); 와 같은 코드

    for (const edge of edges) {
        const [u, v, wt] = edge;
        adj[u].push([v, wt]);
        adj[v].push([u, wt]); 
    }

    return adj;
}

// Returns shortest distances from src to all other vertices
function dijkstra(V, edges, src, target) {
    const adj = constructAdj(edges, V);
    const minHeap = new MinHeap();
    const dist = Array(V).fill(Number.MAX_SAFE_INTEGER);
    const parent = Array(V).fill(-1);  // 부모 노드 추적용

    minHeap.push([0, src]);
    dist[src] = 0;

    while (minHeap.size() > 0) {
        const [d, u] = minHeap.pop();

        // 목표 노드에 도달하면 조기 종료 가능
        if (u === target) break;

        // 이미 처리된 노드라면 스킵
        if (d > dist[u]) continue;

        for (const [v, weight] of adj[u]) {
            if (dist[v] > dist[u] + weight) {
                dist[v] = dist[u] + weight;
                parent[v] = u;  // v의 부모를 u로 설정
                minHeap.push([dist[v], v]);
            }
        }
    }

    // 경로 복원
    const path = [];
    let current = target;
    
    if (dist[target] === Number.MAX_SAFE_INTEGER) {
        return { distance: -1, path: [] };  // 도달 불가능
    }

    while (current !== -1) {
        path.unshift(current);
        current = parent[current];
    }

    return { distance: dist[target], path: path };
}

// Driver code
const V = 5;

// edge list format: [u, v, weight]
const edges = [[0, 1, 4], [0, 2, 8], [1, 4, 6], [2, 3, 2], [3, 4, 10]];

const result = dijkstra(V, edges, 0, 3);

// Print shortest distances in one line
console.log(`Distance: ${result.distance}`);
console.log(`Path: ${result.path.join(' -> ')}`);

```


# Minimum Spanning Tree
*Spanning Tree* 란 어떤 그래프의 spanning tree는 어떤 **그래프의 모든 정점을 포함하고 있는** sub-graph이면서 또한 **tree**이다.
한 그래프에는 여러 스패닝 트리가 있을 수 있다. 
당연히 없을 수도 있다. 연결되지 않은 노드가 있는 disconnected graph에는 스패닝 트리가 존재할 수 없다.

그리고 *Minimum Spanning Tree*란 경로들의 **가중치의 합이 가장 낮으면서** 그래프의 모든 정점을 포함하고 있는 트리다.

## 크루스칼 알고리즘 Kruskal Algorithm

크루스칼 알고리즘을 간단히 설명하면 다음과 같다.
1. 모든 엣지들을 가중치 기준으로 정렬
2. 작은 순서대로 확인해보면서 해당 엣지가 **사이클을 발생시키는지 확인**
3. 발생시키지 않는다면 추가한다.


```js
function kruskalsMST(V, edges) {
    
    // Sort all edges
    edges.sort((a, b) => a[2] - b[2]);
    
    // Traverse edges in sorted order
    const dsu = new DSU(V);
    let cost = 0;
    let count = 0;
    for (const [x, y, w] of edges) {
        
        // Make sure that there is no cycle
        if (dsu.find(x) !== dsu.find(y)) {
            dsu.unite(x, y);
            cost += w;
            if (++count === V - 1) break;
        }
    }
    return cost;
}

// Disjoint set data structure
class DSU {
    constructor(n) {
        this.parent = Array.from({ length: n }, (_, i) => i);
        this.rank = Array(n).fill(1);
    }

    find(i) {
        if (this.parent[i] !== i) {
            this.parent[i] = this.find(this.parent[i]);
        }
        return this.parent[i];
    }

    unite(x, y) {
        const s1 = this.find(x);
        const s2 = this.find(y);
        if (s1 !== s2) {
            if (this.rank[s1] < this.rank[s2]) this.parent[s1] = s2;
            else if (this.rank[s1] > this.rank[s2]) this.parent[s2] = s1;
            else {
                this.parent[s2] = s1;
                this.rank[s1]++;
            }
        }
    }
}

const edges = [
    [0, 1, 10], [1, 3, 15], [2, 3, 4], [2, 0, 6], [0, 3, 5]
];
console.log(kruskalsMST(4, edges));
```

결국 핵심은 해당 엣지가 사이클을 만들어내는지 검사하는 과정에 있는다.
수업에서는 직접 집합을 만들어가면서 예시를 보았는데 (그래서 이해하기가 쉬웠는데), 위에서는 직접 집합을 만드는 방식 대신 `parent`와 `rank`배열을 만들어서 사용한다.

어떻게 위의 코드가 동일한 결과물을 내놓도록 작용하는지 살펴보자.
- `parent` 배열은 각 노드들이 자신이 어느 노드의 집합에 속해있는지 나타낸다. `[0,1,2,3]`에서 `[0,1,2,2]` 가 되면 3번이 2번 노트의 집합에 들어갔다는 뜻.
- `unite()`함수는 두 노드의 집합중에 rank가 낮은 쪽을 높은 쪽에 합친다. 

이렇게 하는 이유는 (당연하게도) 이 방법이 훨씬 효율적이기 때문이다.
이 방법을 사용하면 O(1)의 속도로 집합을 합치는 효과를 낼 수 있기 때문에. (반면에 공간 복잡도는 더 많이 사용한다고 한다.)

### $O(ElogE)$ 시간복잡도
방금 알아본 방법을 통해서 `find()`와 `unite()`함수는 모두 O(1) 의 시간이 걸린다는것을 확인했다. 크루스칼 알고리즘 자체가 정렬 + union/find 이기 때문에 정렬 알고리즘의 시간 복잡도가 곧 크루스칼 알고리즘의 시간복잡도가 된다. 

## 프림 알고리즘 Prim Algorithm
프림 알고리즘은 다익스트라 알고리즘과 흡사하다. 하나의 노드에서 시작해서 그때마다 가중치가 적은 엣지와 연결된 노드를 계속해서 추가해나가는 것.

다른 점은 *거리*에 대한 정의가 다익스트라 알고리즘과 다르다는 점.
- 다익스트라에서의 거리 : 시작점에서 목표 노드까지의 최단경로 총합.
- 프림에서의 거리 : 현재 MST에서 해당 노드까지의 최소 간선 가중치.

### 사이클?
사이클은 이미 연결되어있는 노드들 사이에 엣지가 추가되는것인데, 프림 알고리즘에서는 항상 MST 밖의 노드를 추가해나가기 때문에 사이클이 발생할 수 없다. 

### $O(VlogE)$ 시간복잡도
프림 알고리즘은 다익스트라 알고리즘과 작동 방식이 똑같기 때문에 시간 복잡도도 다익스트라 알고리즘과 같을 수 밖에 없다. 
