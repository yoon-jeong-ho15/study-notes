---
title: You Might Not Need an Effect
date: 2026-01-23
link:
  - https://react.dev/learn/you-might-not-need-an-effect
---
## 이펙트는 탈출구

이펙트는 리액트 밖으로 나와 다른 것들과 리액트 컴포넌트들을 동기화 시켜주는 기능이기 때문에, 외부 시스템과 관련이 없다면 이펙트를 남용하지 않아야 한다. 필요 없는 이펙트는 **가독성**을 떨어뜨리고, **성능**을 낮추고, **에러**를 유발한다.

대표적으로 이펙트를 사용하지 않는 케이스 두 가지 :
- 렌더링에 필요한 데이터 변형하기 - `상태 변경 -> 렌더 -> 커밋 -> 이펙트 -> (또) 상태 변경 -> (또) 렌더 -> ...` 이렇게 불필요한 렌더링 사이클을 한번 더 반복하게 된다. (아래의 '필요 없는 이펙트 제거하기'의 1번 사례 참고)
- 사용자 이벤트 처리

오직 외부 시스템과 동기화 할 때만 사용되어야 한다. 예를들어 jQuery 위젯을 리액트 상태와 동기화 시켜야 하거나, 검색어에 맞게 검색 결과를 가져오거나(데이터 패칭) 하는 경우들.

## 필요 없는 이펙트 제거하기

### 1. props나 state에 따라 state 변경하기

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

우선 `fullName` 이라는 상태를 만들 필요도 없으며, 그리고 그것을 이펙트에서 변경할 필요도 없다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

### 2. 무거운 계산 캐싱하기 : `useMemo`

```Jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

위의 코드는 props 로 받은 `todos`에 `filter`를 적용해서 `visibleTodos`라는 상태를 만들고 있다.
위의 사례와 마찬가지로, 렌더링 중에 계산해낼 수 있는 값은 상태로 관리 할 필요가 없고, 그것을 이펙트에서 변경하는건 더 비효율적이다.

그러나 필터링과 같은 로직은 무거워서 매 렌더링마다 계산하는것이 부담이 될 수 있다. 그래서 위에서 처럼 `useEffect` 안에서  계산해 매 렌더링 마다 계산을 하지 않도록 코드를 작성하는 사례가 있는것이다.

이럴 때 사용하는게 `useMemo` 훅이다. (손수 이 훅을 사용하지 않아도 **리액트 컴파일러**가 자동으로 무거운 계산은 메모이제이션을 적용한다고 한다.)

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

`useMemo` 훅으로 계산한 값은 이 컴포넌트의 props인 `todos`나 `filter`가 변하지 않는 한 다시 계산하지 않는다.
- `useMemo`훅도 렌더링 중에 실행되는 함수이기 때문에 부수효과가 없는 순수 계산으로만 이루어져야 한다.

### 3. prop 변경 시 state 초기화 하기 :  `key` prop

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

위의 코드의 의도는 다른 프로필 페이지로 이동했을 때에도 작성중이던 `comment` 상태가 초기화 되지 않는 버그를 수정하려고 `userId`의 변경에 맞춰 `comment`를 빈 문자열로 업데이트 하려는 것이다.

위 코드의 가장 큰 문제는 불필요한 추가 렌더링을 촉발한다는것. 그리고 `comment` 뿐만 아니라 초기화해야 할 다른 상태들이 더 있다면 그 상태들도 일일이 변경해주어야 한다. 

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

대신 `key` prop을 사용하면 리액트는 해당 컴포넌트를 재사용하지 않고 완전히 새로운 인스턴스를 생성한다(상태들도 초기값으로 전부 초기화 된다). 그래서 상태들이 남아있는 버그가 발생하지 않는다.

### 4. prop 변경에 따라 일부 state 변경하기

`key` 값을 사용해 컴포넌트를 처음부터 만들기 무거운 컴포넌트가 있을 수 있다. 아니면 대부분의 상태는 가만히 두고 일부 상태만 초기화 하거나 변경하고 싶을 수 있다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

3번 사례처럼 상태 초기화를 이펙트에서 하고있으며 이는 잘못된 패턴이다.

**Better**

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

특이하게 이렇게 **이전 렌더의 정보를 저장하는 방법**을 통해 개선할 수도 있다.
`useEffect`의 의존성 배열을 통해 `items`상태가 변경될때만 선택된 아이템을 해제하는 그 방법을 `prevItems`라는 새로운 상태를 도입해 구현했다.

**Best**

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```
