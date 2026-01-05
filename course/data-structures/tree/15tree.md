---
title: 트리 4 (AVL 회전)
date: '2025-08-28'
order: 15
---

AVL 트리는 이전 시간에 봤던것 처럼 일반 이진 검색 트리BST에 '회전'이라는 과정을 통
해 삽입과 삭제 후에 높이의 차이를 k=1 이하가 되도록 조정하는 연산이 추가된 트리다.

# 회전

AVL 조건의 위배?
삽입 혹은 삭제 후에 AVL트리의 조건이 위배되었다고 한다는것은 k가 정확히 2가 되었다는 말이다.

## 삽입 이후 회전

삽입 이후에 AVL 트리의 속성을 회복하려면 삽입 지점에서 시작해 루트까지 이동하면서 **AVL 속성을 만족시키지 않는 첫 번째 노드**가 무엇인지 찾는다.

이 노드를 X라고 하자. 이 때 발생할 수 있는 경우는 총 네 가지다.

1. X의 왼쪽 자식의 왼쪽 서브트리에 노드가 삽입된 경우 (LL)
2. X의 왼쪽 자식의 오른쪽 서브트리에 노드가 삽입된 경우 (LR)
3. X의 오른쪽 자식의 왼쪽 서브트리에 노드가 삽입된 경우 (RL)
4. X의 오른쪽 자식의 오른쪽 서브트리에 노드가 삽입된 경우 (RR)
   1번과 4번의 경우에는 회전 한 번을 통해 완료되지만, 2번과 3번의 경우에는 두 번의 회전을 사용해야 한다.

## 단순 회전

### LL 회전

```
    9
  8
7
```

7이 방급 삽입된 노드라면 X노드는 무엇일까?
8은 1 - 0 = 1 으로 문제가 없고, 9는 2-0 = 2 로 k가 1보다 커져서 9가 첫 노드 X가 된다.
9 노드에서 LL 회전을 수행한다.

```
  8
7   9
```

그런데 각 노드들이 서브트리르 가지는 경우에는 어떡할까?
```
       9
    8      10
  7   8.5
6
```
이렇게 `8.5`와 `10`이 있으면?
`8.5`를 `9`의 왼쪽에 위치시킨다. 
구체적인 코드는 이렇다.
```c
struct AVLTreeNode *SingleRotateRight(struct AVLTreeNode *x) {
	struct AVLTreeNode *w = x->left;
	
	x->left = w->right;
	w->right = x;
	
	x->height = max(height(x->left),height(x->right))+1;
	w->height = max(height(w->left),x->height)+1;
	
	return w;
}
```

1. `w->right`는 w보다 크면서 x보다 작기 때문에 `x->left`로 오면 되고,
2. `w->right`를 x로 변경. (`x->right`와 `w->left`는  변하지 않는다.)
3. 그리고 x와 w의 높이를 업데이트 한다.

### RR 회전
LL과 유사한데 차이점이라면 w대신 y(x의 오른쪽에 있는 알파벳이라)을 사용하고,
`x->left` 대신 `x->right`와 `w->right`대신 `y->left`를 변경한다는 것이다. (정확히 LL이 변경하지 않는 노드를 변경)
```c
struct AVLTreeNode *SingleRotateLeft(struct AVLTreeNode *x) {
	struct AVLTreeNode *y = x->right;
	
	x->right = y->left;
	y->left = x;
	
	x->height = max(height(x->left),height(x->right))+1;
	y->height = max(height(y->right),x->height)+1;
	
	return y;
}
```

> 단순 회전single rotation의 경우에는 LL이나 RR 모두 실제로 중요하게 다뤄지는 서브트리는 LL의 경우 **LR(`w->right`)** 쪽 트리이고 RR의 경우 **RL(`y->left`)** 쪽 트리다. 각 서브트리들을 기존 루트 x에 붙여준다.

## 이중 회전
이중 회전은 RL트리와 LR 트리에 삽입이 발생한 후에 AVL 조건을 위배하게 될 때 실행된다.
'이중double'이라는 말처럼 단일 회전을 두 번 반복하면 되는데 LL의 경우는 RR회전 후에 LL회전, RL의 경우에는 LL회전 후에 RR 회전.
```c
// LR
struct AVLTreeNode *DoubleRotationLeft(struct AVLTreeNode *x){
	x->left = SingleRotateLeft(x->left);
	return SingleRotateRight(x);
}

// RL
struct AVLTreeNode *DoubleRotationLeft(struct AVLTreeNode *x){
	x->right = SingleRotateRight(x->right);
	return SingleRotateLeft(x);
}
```
