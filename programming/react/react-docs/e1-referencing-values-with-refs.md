---
title: Referencing Values with Refs
date: 2026-01-22
link:
  - https://react.dev/learn/referencing-values-with-refs
---
### You will learn

- 컴포넌트에 ref를 추가 하는 방법
- ref값 업데이트 하는 방법
- ref와 state의 차이
- ref 안전하게 사용하는 방법

## Ref란

공식문서에서 상태를 컴포넌트의 메모리라고 한다. 상태가 변할때마다 그에 맞는 최신 상태의 컴포넌트가 다시 그려지기 때문인건데, **어떤 값**들은 컴포넌트가 기억했으면 좋겠지만 변할때마다 새로 렌더링 하지는 않았으면 좋겠으면 하는것들이 있다.

대표적인 예시로 타이머가 있다. 

```jsx
mport { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}

```

스타트 버튼을 누르면 `setInterval`이 실행되면서 10ms 마다 `now` 상태를 업데이트 한다. 그리고 그때마다 `secondsPassed` 값이 계산되고 그걸 화면에 표시한다.

그런데 이걸 멈추려면 `setInterval`이 반환한 id를 사용해 을 멈춰주어야 하는데(`clearInterval(id)`) 이 값은 렌더링과 관련이 없기 때문에 ref로 사용한다. 

## state vs ref
### 타이머 id를 상태로 하면 안되나?

사실 타이머 Id를 상태로 관리해도 정상적으로 작동 한다.

```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const [intervalId, setIntervalId] = useState(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalId);
    setIntervalId(setInterval(() => {
      setNow(Date.now());
    }, 10));
  }

  function handleStop() {
    clearInterval(intervalId);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```

그런데 왜 상태로 하면 안되고 ref로 사용해야할까? 문서에서 이에 대한 답을 찾지는 못했다.

### ref의 내부 원리

리액트 내부 원리는 setter 함수 없이 사용하는 상태와 흡사하다고 한다.

```jsx
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

## 예시 문제

### 1.

```jsx
export default function Chat() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  let timeoutID = null;

  function handleSend() {
    setIsSending(true);
    timeoutID = setTimeout(() => {
      alert('Sent!');
      setIsSending(false);
    }, 3000);
  }

  function handleUndo() {
    setIsSending(false);
    clearTimeout(timeoutID);
  }

  return (
	  //...
```

`handleUndo`를 눌러도 취소되지 않는 이유는 `timeoutID`를 일반 변수로 선언했기 때문이다. 보내기 버튼을 누르면 상태가 변경되면서 컴포넌트가 리렌더링 되는대 일반 변수는 사라진다.  그래서 undo 버튼은 아마 `clearTimeout(null)` 을 호출하게 될거고 그래서 취소되지 않는것.

### 2.

```Jsx
export default function Toggle() {
  const isOnRef = useRef(false);

  return (
    <button onClick={() => {
      isOnRef.current = !isOnRef.current;
    }}>
      {isOnRef.current ? 'On' : 'Off'}
    </button>
  );
}
```

토글 버튼을 눌러도 On과 Off 문구가 바뀌지 않는 이유는, ref는 리렌더링을 촉발하지 않기 때문이다. `console.log(isOnRef.current)`를 찍어보면 버튼을 누를때마다 값은 변하는걸 확인할 수 있다.

### 3.  debouncing

```jsx
let timeoutID;

function DebouncedButton({ onClick, children }) {
  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```

*디바운싱*이란 짧은 순간에 이벤트가 많이 발생할때 마지막 한번만 이벤트를 처리하는 방식을 말한다. 
위에서 버튼을 누를때 마다 **기존 타이머를 지우고 새로운 타이머를 설정**해 1초 뒤에 알림창이 등장한다.
위 코드의 문제는 각 버튼들마다 독립적으로 디바운싱 처리를 하지 못한다는 것인데, 간단히 `let timeoutID` 라고 컴포넌트 밖에서 변수를 선언하고 그걸 사용하고 있기 때문에 모든 버튼들이 하나의 변수를 공유하고 있기 때문이다. 버튼 컴포넌트 안에서 ref를 사용해 해결한다.

### 4. 최신 state 읽기

```jsx
export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  function handleChange(e) {
    setText(e.target.value);
    textRef.current = e.target.value;
  }

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + textRef.current);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={handleChange}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

실제로 이렇게 설계해야 할 경우는 많지 않지만 (Usually, this behavior is what you want in an app. However, there may be occasional cases where you want some asynchronous code to read the latest version of some state.), 위의 코드처럼 최신 상태를 읽기 위해 ref를 사용할 수도 있다.

`alert()` 함수가 호출되면서 전달받은 매개변수는 상태가 변경되어 컴포넌트가 리렌더링 되어도 같은 값을 가지고 있기 때문이다. (리액트는 상태를 변경하지 않고 setter로 아예 새로운 값으로 변경한다.) 



