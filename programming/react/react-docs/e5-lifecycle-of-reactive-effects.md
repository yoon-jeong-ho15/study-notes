---
title: Lifecycle of Reactive Effects
date: 2026-01-26
link:
  - https://react.dev/learn/lifecycle-of-reactive-effects
---
### You will learn

- effect의 생명주기와 컴포넌트의 생명주기의 차이
- 각 effect를 독립적으로 생각하는 방법
- 언제 그리고 왜 effect가 재동기화 해야 하는지
- 어떻게 effect의 의존성이 결정되는지
- 어떤 값이 '반응형 reactive' 이라는 것의 의미
- 비어있는 의존성 배열이 가지는 의미
- 리액트는 어떻게 effect의 의존성 배열이 올바른지 검증하는지
- 린터에 동의하지 않을 때 어떻게 해야하는지

## 이펙트의 생명주기

컴포넌트의 생명주기는 다음과 같다 :

- 마운트 : 화면에 추가
- 업데이트 : 유저 이벤트나 서버와의 통신 등으로 인한 props나 state의 변경이 있을 때 마다 리렌더링
- 언마운트 : 화면에서 제거

그러나 이런 **컴포넌트 생명주기의 틀 안에서 이펙트의 생명주기를 생각하는건 안좋은 방법**이(라고 한)다.

`roomId`라는 prop을 가지는 `ChatRoom` 컴포넌트가 있다고 하자. 이 컴포넌트의 생명주기는 다음과 같은 것이다 :

1. `ChatRoom` 이 마운트 된다. `roomId` 는  `"general"`.
2. `ChatRoom` 이 업데이트 된다. `roomId` 를  `"travel"`로 변경.
3. `ChatRoom` 이 업데이트 된다. `roomId` 를 `"music"`로 변경.
4. `ChatRoom` 언마운트.

이 관점에서 채팅 서버와 연결하는 이펙트의 생명주기를 보면 다음과 같다 :

1. 이펙트가 `"general"` 채팅방과 연결한다.
2. 이펙트가 `"general"` 채팅방과의 연결을 해지하고(클린업) `"travel"` 채팅방과 연결한다.
3. 이펙트가 `"travel"` 채팅방과의 연결을 해지하고  `"music"` 채팅방과 연결한다.
4. 이펙트가 `"music"` 채팅방과의 연결을 해지한다.

마치 컴포넌트가 **리렌더링 될 때 마다 실행되는 콜백**과 같다고 생각이 든다. 하지만 이렇게 생각하기 시작하면 더 복잡해지게 된다. 컴포넌트 관점에서 이펙트를 작성하면 특정 "시점"에 실행되어야 하는지 집착하게 되고 잘못된 코드를 작성하게 될 수 있다.

> [!faq] 무엇이 "복잡해"진다는 걸까?

컴포넌트와 분리해 이펙트의 시작과 중지 사이클에 집중하면 이렇게 된다 :

1. `"general"` 방에 연결된 이펙트 
2. `"travel"` 방에 연결된 이펙트
3. `"music"` 방에 연결된 이펙트

각 이펙트들이 **독립적으로** 언제 연결되고 언제 종료하는지에 대해서만 집중하면 각 연결들이 언제 시작되고 언제 종료되는지에 대해서만 설계하면 된다. 

## 예시 문제

1,2,4 번 문제는 생략.

### 3. stale 값 버그

```jsx
export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [canMove, setCanMove] = useState(true);

  function handleMove(e) {
    if (canMove) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
  }

  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
```

`canMove`를 변경해도 마우스 포인터가 움직이는 이유는 뭘까? 
그 이유는 이펙트에서 참조하는 `handleMove` 함수가 `canMove`가 참일때 정의된 함수이기 때문이다. 자바스크립트의 함수는 *클로저*여서 그렇다. 

그래서 아예 의존성 배열을 삭제하는 방식으로 해결할 수 있다. 그렇게 하면 모든 렌더링마다 이펙트가 실행되면서 최신의 `handleMove`를 사용할 것이기 때문.

```jsx
  useEffect(() => {
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  });
```

하지만 이는 최선의 해결방법이 아니다. 직접적인 연관이 있는 `canMove`상태가 아닌 다른 모든 상태들이 변경될 때 마다 위의 이펙트가 실행되니까.

따라서 다음과 같이 해결하는게 가장 좋은 방법이다.

```jsx
useEffect(() => {
    function handleMove(e) {
      if (canMove) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
    }

    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, [canMove]);
```

왜 `handleMove`에 의존하는 방식이 아니라 **`handleMove`를 비반응형 값으로** 만들고 **`canMove`상태에 의존**하도록 작성할까? `handleMove`는 `canMove`와 상관 없이 렌더링 될 때 마다 바뀌게 되는데, 그럼 위의 코드와 똑같이 결국 모든 렌더링마다 실행하는것이 되기 때문이다. 

### 5. 셀렉트 박스 체인

아래의 코드에서 `planet` 선택시에 `placeList` 를 가져오도록 수정해야 한다.

```jsx
export default function Page() {
  const [planetList, setPlanetList] = useState([])
  const [planetId, setPlanetId] = useState('');

  const [placeList, setPlaceList] = useState([]);
  const [placeId, setPlaceId] = useState('');

  useEffect(() => {
    let ignore = false;
    fetchData('/planets').then(result => {
      if (!ignore) {
        console.log('Fetched a list of planets.');
        setPlanetList(result);
        setPlanetId(result[0].id); // Select the first planet
      }
    });
    return () => {
      ignore = true;
    }
  }, []);

  return (
  //...
```

일단 `planetId`에 의존하는 이펙트 하나를 더 추가할 수 있다. 

```jsx
useEffect(() => {
    if (planetId === '') {
      // Nothing is selected in the first box yet
      return;
    }

    let ignore = false;
    fetchData('/planets/' + planetId + '/places').then(result => {
      if (!ignore) {
        console.log('Fetched a list of places on "' + planetId + '".');
        setPlaceList(result);
        setPlaceId(result[0].id); // Select the first place
      }
    });
    return () => {
      ignore = true;
    }
  }, [planetId]);
```

코드 반복을 피하고 싶다면 커스텀 훅을 사용하면 된다.

```jsx
// 컴포넌트
export default function Page() {
  const [
    planetList,
    planetId,
    setPlanetId
  ] = useSelectOptions('/planets');

  const [
    placeList,
    placeId,
    setPlaceId
  ] = useSelectOptions(planetId ? `/planets/${planetId}/places` : null);

// useSelectOptions 훅  
export function useSelectOptions(url) {
  const [list, setList] = useState(null);
  const [selectedId, setSelectedId] = useState('');
  useEffect(() => {
    if (url === null) {
      return;
    }

    let ignore = false;
    fetchData(url).then(result => {
      if (!ignore) {
        setList(result);
        setSelectedId(result[0].id);
      }
    });
    return () => {
      ignore = true;
    }
  }, [url]);
  return [list, selectedId, setSelectedId];
}
  
```