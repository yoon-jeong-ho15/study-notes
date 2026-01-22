---
title: Referencing Values with Refs
date: 2026-01-22
link:
  - https://react.dev/learn/referencing-values-with-refs
---
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

