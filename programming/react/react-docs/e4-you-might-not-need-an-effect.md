---
title: You Might Not Need an Effect
date: 2026-01-23
link:
  - https://react.dev/learn/you-might-not-need-an-effect
---
### You will learn

- 불필요한 effect를 왜 제거해야 하는지, 그리고 어떻게 제거하는지
- effect 없이 무거운 계산을 캐싱하는 방법
- effect 없이 state를 초기화하고 조정하는 방법
- 이벤트 핸들러간에 같은 로직을 공유하는 방법
- effect가 아닌 이벤트 핸들러에 있어야 하는 로직은 무엇인지
- 부모 컴포넌트에게 변경 사항을 알리는 방법

## 이펙트는 탈출구

이펙트는 리액트 밖으로 나와 다른 것들과 리액트 컴포넌트들을 동기화 시켜주는 기능이기 때문에, 외부 시스템과 관련이 없다면 이펙트를 남용하지 않아야 한다. 필요 없는 이펙트는 **가독성**을 떨어뜨리고, **성능**을 낮추고, **에러**를 유발한다.

대표적으로 이펙트를 사용하지 않는 케이스 두 가지 :
- 렌더링에 필요한 데이터 변형하기 - `상태 변경 -> 렌더 -> 커밋 -> 이펙트 -> (또) 상태 변경 -> (또) 렌더 -> ...` 이렇게 불필요한 렌더링 사이클을 한번 더 반복하게 된다. (아래의 '필요 없는 이펙트 제거하기'의 1번 사례 참고)
- 사용자 이벤트 처리

오직 외부 시스템과 동기화 할 때만 사용되어야 한다. 예를들어 jQuery 위젯을 리액트 상태와 동기화 시켜야 하거나, 검색어에 맞게 검색 결과를 가져오거나(데이터 패칭) 하는 경우들.

## 필요 없는 이펙트 제거하기

다음의 사례들은 리액트를 개발하면서 흔히들 이펙트를 사용해 구현, 해결하려고 하지만 더 효율적이고 적합한 방법이 있는 사례들이다.
즉 아래에 나열된 로직은 구현하는데 이펙트를 사용하고 있다면 매우 높은 확률로 좋지 않은 코드라는 것이다.

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

### 5. 이벤트 핸들러 간 로직 공유

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

두 이벤트 핸들러 모두 `addCart`로 장바구니에 상품을 넣는데, 그 때 마다 알림창을 보여주기 위해서 `product`가 바뀔 때 마다 실행되는 이펙트를 작성했다.

렌더링과 관련이 없고 특정 이벤트에 의해서만 촉발되는 event-specific한 로직이라 이펙트에 작성하는건 적절하지 않다. 어떻게 고쳐야할까? 간단하고 직관적이다. 특정 이벤트에만 실행되는 코드라면 이벤트 핸들러 안에 작성되어야 한다.

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

### 6. POST 요청 보내기

5번 사례와 비슷한 내용이라서 생략.
같은 post 요청이지만 어떤 요청은 이펙트에 작성해야 하고 어떤 요청은 이벤트 핸들러에 작성해야 하는가에 대한 설명. 
특정 이벤트에 종속적이라면 이벤트 헨들러 안에서 요청을 보내야한다.

### 7. 연쇄 계산

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

이 코드의 의도는 어떤 상태(`card`) 변했을 때 그 상태의 변화에 기반해 다른 상태를(`goldCardCount`, `round`, `isGameOver`) 변경하려는 것이다.
이렇게 이펙트로 상태를 연쇄적으로 변경한다면 불필요한 리렌더링이 여러번 발생하게 된다 : card 변경 -> 리렌더링 -> goldCardCount 변경 -> 리렌더링 ->  ....

state에 대한 다른 문서에서도 보았지만, 다른 상태에 의해서 **계산될 수 있는 값**은 굳이 상태로 관리하지 않는것이 좋고, 카드를 확인하는 **이벤트에 종속**적인 로직들이니까 이벤트 핸들러에서 실행되어야 한다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount < 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```


### 8. 애플리케이션 초기 시작

애플리케이션을 처음에 열었을때 실행되는 로직들이 있다. 세션정보나 로컬 스토리지에서 데이터를 가져오는 등의 로직들. 
그렇다면 이 로직들은 루트 컴포넌트의 이펙트에 작성해야 할까? 루트 컴포넌트가 렌더링 된다는게 곧 애플리케이션을 처음 실행한다는 뜻이니까. 

이펙트에 작성한다면 마운트당 한번씩 실행되지 않도록 최상위 플래그 변수를 사용한다.

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

혹은 최상위 레벨에 작성할 수도 있다. 최상위 레벨의 코드는 컴포넌트가 **렌더링 되기 이전에, import 할 때** 미리 실행한다.

```jsx
if (typeof window !== 'undefined') { // Check if we're running in the browser.
   // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

### 9. state 변경을 부모에게 알리기

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

`isOn` 상태가 변경될 때 부모에게 알리기 위해 `onChange` 함수를 이펙트 안에서 실행하고 있다. 이렇게 하면 다음과 같은 비효율적인 과정을 지난다 : isOn 변경 -> 리렌더링 -> onChange 호출 -> 부모의 상태 변경 -> 부모 리렌더링.

문제를 해결하기 위해서는 **이벤트 핸들러에서 `onChange`를 호출**하는 방법이 있다.

```jsx
  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }
```

이렇게 하면 `setIsOn`이 촉발한 리렌더링과 `onChange`가 촉발한 리렌더링이 리액트의 배칭 처리 방식에 의해 한번에 일어나서 더 효율적이다.

또 다른 방법으로는 **"Lifting state up" 방식을 사용**해 `isOn`도 prop으로 건내주고 부모 컴포넌트에서 관리하는 것. 

### 10. 부모에게 데이터 전달하기

이 사례도 바로 앞의 9번 사례와 비슷하다. 본질적으로 자식 컴포넌트에서 이펙트를 통해 부모 컴포넌트의 상태를 변경하려고 한다. 
9번 사례에서는 렌더링 효율성에 대해 이야기했다면 여기는 추가적으로 **리액트의 데이터 흐름**을 거스른다는 문제점에 대해 이야기한다. 리액트에서는 언제나 부모에서 자식으로 데이터가 흘러가야 한다. 만약 반대 방향의 설계가 중간에 섞여있다면 문제가 발생했을때 원인을 찾아내기 힘들어진다.

### 11. 외부 저장소 구독 : `useSyncExternalStore`

외부 저장소란 리액트가 아닌 브라우저 API나 외부 라이브러리(zustand 등)를 말한다. 어떤 컴포넌트는 리액트 state 대신 다른 곳에서 데이터를 가져와야 할 필요가 있을 수 있다.

```jsx
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

위의 사례를 보면 커스텀 훅을 만들어서 `navigator.onLine` 이라는 브라우저 API를 통해 데이터를 가져오고 있다. 
리액트에서는 이 용도에 맞는 전용 훅을 사용하기를 권장한다. 
[useSyncExternalStore 에 대해](https://react.dev/reference/react/useSyncExternalStore)

> [!faq] "**외부 시스템과의 동기화**"라는 이펙트의 목적에 알맞는 사용법으로 보이는데 무엇이 문제일까?
> 

### 12. 데이터 패칭

이 사례는 이펙트를 제거해야 하는 사례가 아니라, 이펙트 사용시의 주의점에 대해 이야기 하고 있다. [레이스 컨디션을 예방 하기 위해 클린업 함수를 설정해야 한다.](https://react.dev/learn/synchronizing-with-effects#fetching-data)

## 예시 문제

### 1. 이펙트 없이 데이터 변경하기

```jsx
export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => {
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => {
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => {
    setFooter(
      <footer>
        {activeTodos.length} todos left
      </footer>
    );
  }, [activeTodos]);
  
  //...
```

`activeTodos`와 `visibleTodos` 모두 렌더링 중에 계산 가능한 값이기 때문에 상태로 관리 할 필요가 없으며, 그래서 이펙트도 필요가 없다.
`footer`도 어차피 `activeTodos`의 변경과 함께 리렌더링 되면서 최신 값을 가지기 때문에 이펙트를 사용할 필요가 없다.

### 2. 이펙트 없이 캐싱하기

```jsx
export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }
```

`visibleTodos`는 계산 가능한 값이지만, 모든 렌더링마다 계산될 필요는 없다. 그리고 데이터가 많은 경우 계산이 무거워지기 때문에 `useEffect`를 통해 의존하지 않는 상태의 변경에는 계산을 실행하지 않도록 작성한 코드다. 
하지만 이펙트를 사용해 상태를 변경하는건 안티패턴이기 때문에 이럴때 사용하는`useMemo` 훅을 사용해 계산된 값을 기억한다.

```jsx
  const visibleTodos = useMemo(
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );
```

사실 더 간단한 방법이 있는데 `text` 상태를 관리하는 자식 컴포넌트를 만드는 것이다(1번 문제의 예시 코드처럼). 그러면 텍스트 입력시마다 리렌더링 되지 않으니 `useMemo` 훅 없이 일반적인 변수로 `visibleTodos`를 관리해도 같은 효과를 낸다.

### 3. 이펙트 없이 상태 초기화

```Jsx
export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);
  
  //...
```

`key` prop을 사용해서 이펙트 없이 상태를 초기화 해야 하는데 `key` prop을 주기 위해 컴포넌트 계층을 나눠야 한다.

```jsx
export default function EditContact(props) {
  return (
    <EditForm
      {...props}
      key={props.savedContact.id}
    />
  );
}

function EditForm({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
  //...
```

