---
title: Removing Effect Dependencies
date: 2026-01-28
link:
  - https://react.dev/learn/removing-effect-dependencies
---
이펙트는 의존성 배열에 있는 값이 변할 때 마다 실행되기 때문에, 불필요한 의존성을 가진다면 이펙트가 너무 자주 실행되거나 무한 루프에 빠질 수 있다.
그래서 무엇이 진짜 의존성인지, 무엇이 불필요한 의존성인지 구분해야 한다.

## 의존성은 선택할 수 없다.

개발자가 이펙트가 어떤 의존성을 가지는지 선택할 수 없다.
이펙트 안에서 사용되는 모든 반응형 값 reactive value(state, props, 컴포넌트에서 선언된 변수)은 반드시 의존성 배열에 들어가야 한다.
그래서 어떤 의존성을 제거하고싶다면 해당 값을 비반응형 값으로 변경하거나, 해당 값을 사용하지 않거나 하는 방법으로 문제를 해결해야 한다.

## 의존성 제거하기

이펙트가 너무 자주 실행되거나, 일부 의존성 값에 '반응'하지 않으면서 최신 값을 사용하고 싶거나 할 때 다음과 같은 체크리스트를 확인해보는것이 도움이 된다.

### 1. 이벤트 핸들러로 옮겨야 하지 않을까?

만약 이펙트의 어떤 코드가 특정 유저 이벤트에 종속적이라면? " You Might not need an Effect" 페이지에서 봤던 것 처럼 이 경우는 이펙트가 필요하지 않고 이벤트 핸들러에 포함되어야 한다.

### 2. 이펙트가 여러가지 일을 하는것 아닐까?

하나의 이펙트에서 여러가지 일을 하는 경우 의존성이 늘어나고 따라서 서로 관련 없는 코드끼리 서로의 의존성에 의해 불필요하게 실행될 수 있다.

### 3. 다음 state를 계산하기 위해 state를 읽는가?

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
  // ...
```

이렇게 작성하면 메시지가 도착할 때 마다 이펙트가 재실행 된다. `message` 상태를 직접 사용하고 의존중이기 때문이다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이렇게 값을 직접 변경하지 않고 *업데이터 함수*를 통해 상태를 변경하면 된다.

### 4. 값의 변화에 '반응'하지 않으면서 값을 읽고싶은가?

`useEffectEvent`훅 내용 참고.

prop으로 받은 이벤트 핸들러를 이펙트 안에서 사용하고 싶을때에도 적용된다. 

```jsx
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ All dependencies declared
  // ...
```

`onReceiveMessage` 는 함수이기 때문에 모든 렌더링마다 주소값이 달라 이펙트는 매번 변경되었다고 판단하고 이펙트를 실행한다. 
(굳이 prop으로 받은 함수가 아니라 컴포넌트 안에서 정의된 함수도 마찬가지다. 하지만 이러한 경우에는 함수를 이펙트 안에 정의하면 해결 가능하다. )

```jsx
  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
```

### 5. 어떤 반응형 값이 의도치 않게 변경되는가?

```jsx
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };
  
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
```

위 코드의 의도는 `serverUrl`이나 `roomId`가 변경되어 `option`객체가 업데이트 되면 이펙트를 실행하는 것이지만, 함수와 마찬가지로 객체도 모든 랜더링마다 다른 주소값을 가지면서 모든 리렌더링마다 채팅방 연결이 반복된다.

문서에서는 두 가지 방법은 제안한다. 만약 의존하고있는 객체가 **정적**이라면, 변하지 않는다면, 그것을 **컴포넌트 밖에서 정의**하는 것이다.

그러나 이 예시처럼 `roomId`가 변경된다면 이 객체는 정적인 객체가 아니라 **동적**인 객체가 된다. 이럴땐 어떻게 해야할까? 이렇게 `option` 객체를 **이펙트 안에서 정의**하면 된다.

```jsx
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
```

그런데 **객체가 prop인 경우**는 어떻게 해야할까? `option` 객체를 부모 컴포넌트로부터 전달받는다면? 

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
```

`roomId`와 `serverUrl`을 추출해서 사용하면 된다.

## 예시 문제

### 1. 인터벌 초기화 문제

```jsx
  useEffect(() => {
    console.log('✅ Creating an interval');
    const id = setInterval(() => {
      console.log('⏰ Interval tick');
      setCount(count + 1);
    }, 1000);
    return () => {
      console.log('❌ Clearing an interval');
      clearInterval(id);
    };
  }, [count]);
```

`setCount`에서 직접 값을 설정하지 않고 업데이터 함수를 사용하면 된다.

### 2. 애니메이션 재촉발 문제

```jsx
function Welcome({ duration }) {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(duration);
    return () => {
      animation.stop();
    };
  }, [duration]);
```

최신값을 읽을 수 있지만 이펙트가 반응하지 않게 만들어주는 `useEffectEvent`훅을 사용하면 된다.

```jsx
  const onAppear = useEffectEvent(animation => {
    animation.start(duration);
  });

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    onAppear(animation);
    return () => {
      animation.stop();
    };
  }, []);
```

### 3. 재연결 되는 채팅

```jsx
export default function ChatRoom({ options }) {
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
```

prop으로 받은 `option`에서 필요한 값을 추출해서 사용해야 한다.

```jsx
export default function ChatRoom({ options }) {
  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
```

### 4. 재연결 되는 채팅 2

```jsx
export default function ChatRoom({ roomId, createConnection, onMessage }) {
  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [createConnection, onMessage]);
```

여기서 `roomId`와 `isEncrypted`가 변경될 때만 새로 연결하고 싶다면? 일단 `createConnection`과 `onMessage`를 의존성에서 제거해야 한다.
`onMessage`의 경우 이펙트 이벤트를 사용하면 된다.

```jsx
export default function ChatRoom({ roomId, createConnection, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage);

  useEffect(() => {
    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    // ...
```

그런데 `createConnection`의 경우는 복잡하다. `roomId`와 `isEncrypted`에 **반응해야 한다.**
즉 위의 두 state를 부모로부터 받아와야 한다는 것이다.

```jsx
      <ChatRoom
        roomId={roomId}
        isEncrypted={isEncrypted}
        onMessage={msg => {
          showNotification('New message: ' + msg, isDark ? 'dark' : 'light');
        }}
      />
```

그리고 `createConnection`이라는 함수는 부모에서 정의하지 않고 `Chatroom` 컴포넌트에서 정의한다. 하지만 **컴포넌트 안에서 정의한 함수도 마찬가지로** 렌더링마다 주소값이 바뀐다. 그러므로 단순히 컴포넌트 안에서 정의해서는 안되고 이펙트 안에 정의해야 한다.

```jsx
  useEffect(() => {
    function createConnection() {
      const options = {
        serverUrl: 'https://localhost:1234',
        roomId: roomId // Reading a reactive value
      };
      if (isEncrypted) { // Reading a reactive value
        return createEncryptedConnection(options);
      } else {
        return createUnencryptedConnection(options);
      }
    }

    const connection = createConnection();
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, isEncrypted]); // ✅ All dependencies declared
```