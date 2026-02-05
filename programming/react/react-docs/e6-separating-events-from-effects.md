---
title: Separating Events from Effects
date: 2026-01-21
link:
  - https://react.dev/learn/separating-events-from-effects
---
### You will learn

- 이벤트 핸들러와 effect 중에 선택하는 방법
- effect가 반응형이고 이벤트 핸들러는 그렇지 않은 이유
- effect의 일부 코드가 반응형이지 않길 바랄 때 할 수 있는 방법들
- effect event를 정의하고 effect 에서 분리하는 방법
- effect event를 통해 effect에서 최신의 props나 state를 읽는 방법

## 이벤트 핸들러 vs 이펙트

이벤트 핸들러는 말 그대로 사용자 이벤트에 의해 촉발되어서 실행되는 코드고, 이펙트는 *동기화 synchronization*이 필요할 때 마다 실행되는 코드다.

### Reactive values 반응형 값

Props, State, 그리고 컴포넌트 안에서 정의된 변수들을 **반응형 값 reactive value** 라고 한다. 이 세가지는 컴포넌트가 리렌더링 될 때 마다 변하기 때문이다.

이벤트 핸들러와 이펙트를 구분하는 기준도 이 반응형 reactive 여부다. 

이벤트 핸들러 안의 로직은 반응형이 아니다. `sendMessage(message)` 에서 `message` 상태가 변하더라도 실행되지 않지만, 실행될 때에는 변한 상태를 읽을 수 있다. (Event handlers can read reactive values without “reacting” to their changes.)

반면 이펙트의 로직은 반응형이다. 즉 반응형 값들(속성, 상태, 컴포넌트 안에서 정의된 변수)이 변경되면 함께 실행된다. 

## 이펙트에서 비반응형 로직 제거하기

따라서 이펙트 안에는 이펙트가 의존하고 있는 값에 반응하는 **반응형 로직들만 존재해야 한다.**

하지만 까다로운 경우가 존재하는데, 가령 채팅서버 입장시 채팅방 입장을 알리는 알림을 표시하고 싶다면 자연스럽게 `useEffect`안에 작성하게 될 것이다.

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

사용자가 설정한 ui 테마에 맞는 알림창을 띄우기 위해 `theme` 속성을 `showNotification` 함수의 인자로 전달하는데, 이렇게 작성하는 경우 `theme` 을 **의존성 배열에 넣어야 한다**.

그런데 이렇게 하면 사용자가 `theme`을 변경할 때 마다 이펙트가 실행되면서 채팅 서버와 다시 연결되고 알림창이 또 표시될것이다. 어떻게 `theme`을 의존성 배열에 넣지 않으면서 채팅방 입장시에만 알림창을 표시할 수 있을까?

### Effect Event

이런 경우를 위해 개발된 훅 `useEffectEvent`가 있어서 이걸 사용하면 된다.

```Jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

이렇게 하면 `showNotification` 함수는 최신상태의 `theme`을 사용할 수 있고, `useEffect` 는 정말로 필요한 값에만 의존할 수 있다.

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
  // ...
}
```

위와 같은 경우에도 사용자가 페이지에 방문할 때 마다 장바구니 정보와 함께 로그 기록을 남길 수 있다.

### Effect Event 사용시 주의사항

- **이펙트 안**에서만 사용한다.
- **다른 컴포넌트나 훅**에 전달하지 않는다.

```Jsx
function Timer() {
  const [count, setCount] = useState(0);

  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 Avoid: Passing Effect Events

  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  useEffect(() => {
    const id = setInterval(() => {
      callback();
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay, callback]); // Need to specify "callback" in dependencies
}
```

여기서는 `onTick`이라는 이펙트 이벤트를 정의하고 `useTimer`라는 훅에 전달하고 있다. 그래서 훅의 이펙트에서 `callback`이라는 속성을 받아 사용하고 있기 때문에 의존성 배열에 추가해야 하는데, 이는 쓸모없는 의존성을 추가하는 것이라 좋지 않은 설계이다.

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  useTimer(() => {
    setCount(count + 1);
  }, 1000);
  return <h1>{count}</h1>
}

function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ Good: Only called locally inside an Effect
    }, delay);
    return () => {
      clearInterval(id);
    };
  }, [delay]); // No need to specify "onTick" (an Effect Event) as a dependency
}
```

이렇게 `callback`으로 외부에서 함수를 주입받더라도 훅 안에서 이펙트 이벤트를 정의해야 한다. 

## 예시 문제

### 2. 잠깐 멈추는 카운터

```jsx
export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + increment);
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, [increment]);

```

이펙트가 다시 실행되면서 타이머가 새롭게 설정되기 때문에 `increment`를 조정할 때 잠깐 프리징되는 현상이 발생한다.

```jsx
  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, 1000);
    return () => {
      clearInterval(id);
    };
  }, []);
```

이렇게 `increment` 의존성을 제거하고 `useEffectEvent` 훅을 사용하면 된다.

### 3. 딜레이 설정이 적용 안되는 카운터

```jsx
export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);
  const [delay, setDelay] = useState(100);

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  const onMount = useEffectEvent(() => {
    return setInterval(() => {
      onTick();
    }, delay);
  });

  useEffect(() => {
    const id = onMount();
    return () => {
      clearInterval(id);
    }
  }, []);
```

인터벌 딜레이를 변경해도 적용이 되지 않는다. 적용이 안되는 이유는 이펙트가 `delay` 값에 반응하지 않기 때문이다.

```jsx

  const onTick = useEffectEvent(() => {
    setCount(c => c + increment);
  });

  useEffect(() => {
    const id = setInterval(() => {
      onTick();
    }, delay);
    return () => {
      clearInterval(id);
    }
  }, [delay]);
```

`onMount` 의 로직은 이펙트가 가져야하는 반응형 로직이다. 

### 4. 알림창 딜레이

```Jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Welcome to ' + roomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected();
      }, 2000);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

```

채팅방을 general -> travel -> music 로 빠르게 두 번 변경하면 알림창은 music 알림창 두 개가 뜬다. travel 하나 music 하나가 뜨도록 수정해야 한다.

이건 반대로 이펙트 이벤트로 분리하면서 최신 `roomId`를 가져와서 발생하는 문제다.
하지만 이펙트 안에 넣어서 문제를 해결하려면 `theme`도 의존성 배열에 들어가야 해서 불가능.
`onConnected`가 최신 `rooomId`를 읽지 못하게 하려면 어떻게 해야할까? 보니까 정답은 직접 `onConnected`함수에 인자로 건내주는 것이다.

```jsx
  const onConnected = useEffectEvent(connectedRoomId => {
    showNotification('Welcome to ' + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      setTimeout(() => {
        onConnected(roomId);
      }, 2000);
    });
```

그런데 추가로 생각해보면 과연 알림창을 굳이 두 번 보여줄 필요가 있을까? 
문제에서도 추가로 *디바운스*를 적용해 마지막 접속 채널에 대한 알림만 보여주는 방식을 소개한다.

```jsx
 useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    let notificationTimeoutId;
    connection.on('connected', () => {
      notificationTimeoutId = setTimeout(() => {
        onConnected(roomId);
      }, 2000);
    });
    connection.connect();
    return () => {
      connection.disconnect();
      if (notificationTimeoutId !== undefined) {
        clearTimeout(notificationTimeoutId);
      }
    };
  }, [roomId]);
```

