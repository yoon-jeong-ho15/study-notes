---
title: Reusing Logic with Custom Hooks
date: 2026-01-28
link:
  - https://react.dev/learn/reusing-logic-with-custom-hooks
---
### You will learn

- 커스텀 훅이란 무엇이고 어떻게 작성하는지
- 컴포넌트 간 로직을 공유하는 방법
- 커스텀 훅 명명하는 방법과 구조를 설계하는 방법
- 언제 왜 로직을 커스텀 훅으로 분리해야 하는지

## '커스텀 훅'이란?

커스텀 훅은 두 개 이상의 컴포넌트에서 똑같은 리액트 훅 코드를 사용할 때 그걸 다른 파일로 분리해 재사용 할 수 있게 해준다. 

```jsx
const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
```

위 코드를 보면 네트워크와 관련된 브라우저 API를 사용해 `isOnline` 상태를 변경해주는 코드다. 이 똑같은 기능을 `SaveButton`이라는 컴포넌트와 `StatusBar` 컴포넌트에서 **똑같이 사용할 때** 이 코드를 커스텀 훅으로 분리할 수 있다.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

```

## 커스텀 훅 명명 규칙 `use`

리액트의 네이밍 컨벤션

- 컴포넌트 : 대문자로 시작, 카멜케이스. `StatusBar`, `SaveButton`, `HomePage`, `App` 등
- 훅 : `use`로 시작, 그 이후로는 대문자로 시작하는 카멜케이스. `useOnlineStatus`, `useState`, `useWindowSize` 등

## state 자체를 공유하는게 아니다.

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  // ...
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  // ...
}
```

위의 코드는 아래와 완전히 동일하게 작동한다.

```jsx
function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}

function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    // ...
  }, []);
  // ...
}
```

커스텀 훅을 사용했을때 할 수 있는 오해 
- `useOnlineStatus` 에서 `isOnline`이라는 상태가 관리되는 것이고 
- 그 state를 각 컴포넌트가 가져와 사용하는것이다.
- 즉 **state는 하나만 존재**하고 그걸 사용하는 **컴포넌트가 여럿**이다.

그러나 똑같은 코드를 각 컴포넌트에서 사용하는것과 똑같이 작동한다는 것을 기억해야 한다. 각 컴포넌트에서 각자 `useState`와 `useEffect`를 사용해 각자의 `isOnline` 컴포넌트를 관리하는것처럼, 커스텀 훅을 사용한다는것은 **상태를 관리하는 로직을 공유**한다는것이지 **상태 자체를 공유하는것이 아니다**.

## 훅 끼리의 반응형 값 전달

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

위의 코드를 보면 `useChatRoom`은 이펙트만 가지고 있다. `useEffect`와 `useState`가 서로 state를 주고 받으면서 작동할 수 있었듯 커스텀 훅인 `useChatRoom`과 `useState`도 같이 작동할 수 있다. prop인 `roomId`도 마찬가지.

## 커스텀 훅에 이벤트 핸들러 전달하기

지금은 커스텀 훅에 알림창을 띄우는 함수가 하드코딩 되어있다.

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

만약에 컴포넌트마다 약간 다르게 알림창을 띄우고 싶다면? 그렇다면 `useChatRoom`의 하드코딩된 부분을 컴포넌트가 전달해줘야 한다. 그래야 커스텀 훅은 재사용 가능하면서 각 컴포넌트마다 각자 원하는 이벤트 핸들링 로직을 사용할 수 있기 때문이다.

컴포넌트로부터 이벤트 핸들러를 전달 받으면 간단하지만, 이펙트에서 배웠던대로 함수는 렌더링마다 다른 주소값을 가지기 때문에 의존성 배열에서 빼야한다.

```jsx
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
}
```

## 언제 커스텀 훅을 사용할까

반복된 코드가 있을때마다 커스텀 훅으로 분리 할 필요는 없지만, 이펙트를 작성해야할 때마다  커스텀 훅으로 분리할 것을 고려해보라 한다.(재사용 되지 않는다 하더라도 분리해야 할까?) 이펙트는 오직 리액트 외부에 접근할때만 사용하기 때문에 이펙트 코드를 컴포넌트에서 분리하면 훨씬 가독성이 높아지기 때문이다.


## 예시 문제

### 3. 커스텀 훅 분리하기

```jsx
export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, delay);
    return () => clearInterval(id);
  }, [delay]);
  return count;
}
```

이 훅을 두 개의 훅으로 분리하라고 한다. `useCounter`는 주어졌고 `useInterval`을 작성해야 한다.

```jsx
export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

답은 아래와 같다.

```Jsx
export function useInterval(onTick, delay) {
  useEffect(() => {
    const id = setInterval(onTick, delay);
    return () => clearInterval(id);
  }, [onTick, delay]);
}

```

나는 `useEffectEvent`를 사용했는데 답안에서는 `onTick`도 의존성 배열에 추가했다. 왜 그런지 의아했는데 바로 다음 문제가 이와 관련된 문제라고 한다.

엄밀하게 모든 렌더링마다 이펙트가 실행되어도 문제는 없다. 왜냐하면 `setInterval`이 계속 새로 설정되어도 정확히 1초 이후에 counter가 증가하는것은 똑같으니까.

### 4. 

```jsx
export default function Counter() {
  const count = useCounter(1000);

  useInterval(() => {
    const randomColor = `hsla(${Math.random() * 360}, 100%, 50%, 0.2)`;
    document.body.style.backgroundColor = randomColor;
  }, 2000);
```

```jsx
export function useCounter(delay) {
  const [count, setCount] = useState(0);
  useInterval(() => {
    setCount(c => c + 1);
  }, delay);
  return count;
}
```

이렇게 각자 다른 `delay`값을 가지고 `useInterval`을 호출하고 있다.
`useInterval`이 위에서처럼 `[onTick, delay]`를 의존성 배열로 가지고 있는 경우 발생하는 문제는 배경 색이 변하지 않는것이다.

이유는 무엇일까? 1초 마다 `count`값이 업데이트 되면서 `Counter` 컴포넌트가 리렌더링 된다. 그 결과 `useInterval()`을 호출하면서 넘겨준 색을 변경하는 화살표 함수(`useInterval` 에서 `onTick`이라는 매개변수로 전달 받는 함수)가 다른 주소값을 가지게 되고, 그 결과 색이 바뀌는 2초가 지나기 전에 계속해서 타이머가 리셋되기 때문에.

```Jsx
export function useInterval(callback, delay) {
  const onTick = useEffectEvent(callback);
  useEffect(() => {
    const id = setInterval(onTick, delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

그래서 이렇게 이펙트 이벤트를 사용해야 한다.

### 5. 엇갈린 움직임 구현

마우스를 따라다니는 점이 5개가 있음에도 불구하고 모두 동시에 움직이기 때문에 하나로 보인다. 

```jsx
function useDelayedValue(value, delay) {
  // TODO: Implement this Hook
  return value;
}

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos3, 50);
  return (
    <>
      <Dot position={pos1} opacity={1} />
      <Dot position={pos2} opacity={0.8} />
      <Dot position={pos3} opacity={0.6} />
      <Dot position={pos4} opacity={0.4} />
      <Dot position={pos5} opacity={0.2} />
    </>
  );
}
```

```jsx
export function usePointerPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);
  return position;
}
```

