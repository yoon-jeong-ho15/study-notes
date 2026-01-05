---
title: 트리 3 (균형 이진 검색 트리)
date: '2025-08-27'
order: 14
---

# 삭제

이진 검색 트리에서 노드의 삭제 연산은 다른것보다 복잡하다.
왜냐하면 삭제한 노드가 자식이 있는 경우 그 자식들을 버리지 않으면서 다시 재배치 해야 하기 때문.

```
         8
    4        12
  2   6    10   14
 1 3 5 7  9 11 13 15
0
```

(위에는 어떤 이진 트리일까? 완전complete 이진 트리.)

어떤 노드를 삭제할 때에는 세 가지 경우가 있을것이다.

1. 자식 노드가 없는 경우
2. 자식 노드가 하나 있는 경우
3. 자식 노드가 둘 있는 경우

## 자식 노드 없음

`node.left`와 `node.right` 모두가 null 인 경우
위의 트리에서 9를 지우려고 한다면
그 부모인 10에서 `left`를 null로 바꿔준다.

## 자식 노드 1개

`node.left` 만 있는 경우
위의 트리에서 1을 지우려면 1의 부모인 2와 1의 자식인 0을 이어줘야 한다.

## 자식 노드 2개

위의 트리에서 4를 지우려면?
8의 left에 2나 6중에 하나를 넣어야하나? 그런데 이렇게 할 수 없는게 2와 6 모두 자식 노드를 가지고 있기 때문에 꼬여버린다.

그래서 먼저
지우려고 하는 노드의 값 보다 큰 것 중에 가자 작은것 (후임노드)를 찾는다. 여기서는 5.
그리고 (일단) 이 5를 8의 left로 연결해준다.

```
          8
    5         12
  2   6
 1 3 5 7
0
```

이렇게 되면 문제가 5가 중첩된다는 점.
이걸 해결하기 위해 다시 5의 오른쪽 서브트리에서 5와 일치하는 노드를 지워주면 된다.

## js 코드

```js
    remove(data) {
        this.root = this.removeNode(this.root, data);
    }

	removeNode(node, data) {
        // 1.
        if (!node) return null;

        // 2. 노드 찾기
        if (data < node.data) {
            node.left = this.removeNode(node.left, data);
            return node;
        }
        if (data > node.data) {
            node.right = this.removeNode(node.right, data);
            return node;
        }

        // 3. 노드 찾음
        // 1) 자식 0
        if (!node.left && !node.right) {
            return null;
        }

        // 2) 자식 1
        if (!node.left) {
            return node.right;
        }
        if (!node.right) {
            return node.left;
        }

        // * 3) 자식 2
        const temp = this.findMin(node.right);
        node.data = temp.data;

        node.right = this.removeNode(node.right, temp.data);
        return node;
    }

    findMin(node) {
        if (!node.left) {
            return node;
        }
        return this.findMin(node.left);
    }
```

# 균형 이진 트리 Balanced

일반 BST가 가지는 큰 단점이 있다.
바로 데이터를 입력할 때 아무렇게나 마구 입력하는 경우 매우 비 효율적인 구조로 만들어질 수 있다.
`[1,2,3,4,5]` 이 순서대로 자료를 입력하면 일반 링크드 리스트와 다를바가 전혀 없다.

그래서 이렇게 경사트리가 만들어지는걸 방지하기 위해 고안된 것이 균형 이진 트리.
그 중 대표적인 것이 AVL 트리다.

## AVL 트리의 속성

각 노드 왼쪽 서브트리와 오른쪽 서브트리의 높이 차이를 k(**balance factor**)라고 하고, 이 k가 1 이하이여야 한다.
이렇게 되면 항상 $O(logn)$정도의 시간 복잡도를 통해 트리 데이터를 조회, 수정을 할 수 있다.

기본적으로 이진 트리와 크게 다른것은 없는데, 이진트리에서 자료를 **삽입**하고 **삭제**할 때 경사 트리가 만들어지기 때문에 삽입과 삭제 과정에 차이가 있다.

그리고 원래 한 노드에는 `data, left, right`라는 멤버가 있는데 여기에 `height`라는 높이를 알 수 있는 속성도 추가한다고 한다.

## 회전

삽입 혹은 삭제 후에 AVL의 조건 (k가 1 이하일 것)이 위배된다면 '회전'이라는 과정을 통해 트리를 다시 정렬한다.
이에 대해서는 다음 시간에 다룬다.
