---
title: Synchronizing with Effects
date: 2026-01-21
link:
  - https://react.dev/learn/synchronizing-with-effects
---
## 이펙트란?

리액트 컴포넌트 안에는 두 종류의 코드가 있다.

- 렌더링 코드 : 순수 함수
- 이벤트 핸들러 : side effect 유발

그런데 이 두가지로 부족한 경우가 있다. 채팅방 입장과 같이 컴포넌트가 **렌더링 될 때 실행되어야 하는 코드**도 있다. 순수 함수가 아니기 때문에 렌더링 코드에 포함될 수 없고, 딱히 서버에 연결할 이벤트가 있는것도 어색하다.

이럴때 사용하는게 Effect다. 렌더링도 하나의 이벤트라면 렌더링이라는 이벤트 핸들러라고 볼 수 있을까.

이펙트의 주 사용 목적은 리액트 밖의 외부 시스템(브라우저 DOM api, 네트워크, 타 라이브러리 등)과의 소통이다. 그래서 이러한 목적이 아닌 경우에는 사용하지 않는것이 좋다.

## 이펙트 작성하기

### 1. 이펙트 정의

- 컴포넌트의 최상위 레벨에 작성한다.
- 이펙트는 화면이 업데이트 된 후에 실행되기 때문에 약간의 딜레이가 발생할 수 있다.
- `useEffect` 훅 안에 작성해야 하는 이유는 먼저 렌더링 코드의 순수성을 방해하지 않기 위함이기도 하지만, 렌더링 이전에는 이펙트에서 사용할 DOM 노드가 존재하지 않기 때문이다.
### 2. 의존성 배열 

기본적으로 이펙트는 모든 렌더링마다 실행되는데, 많은 경우 성능 저하를 유발하거나(키보드 입력마다 채팅 서버에 연결), UI를 망가뜨린다(키보드 입력마다 fade-in 애니메이션).
이를 방지하기 위해 의존성 배열에 이펙트 실행을 유발하는 변수들을 넣어 이펙트 실행을 특정할 수 있다.
### 3. 클린업 함수

이펙트는 리액트 외부와 소통하는 창구이기 때문에, 반드시 리액트 컴포넌트와 생명주기가 같지 않다. 그래서 이벤트 리스너 등록, 외부 라이브러리의 인스턴스 생성, 웹소켓 연결과 같은 코드들은 리액트 컴포넌트가 사라져도(언마운트) 계속 남아있게 되기 때문에 이것을 해지해주는 클린업 함수가 필요하다.

### ref 는 의존성 배열에 포함되지 않는다

Ref 객체는 리액트에서 별도로 보관하는 객체이기 때문에 교체되지 않는다. 그래서 상태와 달리 렌더링 사이에도 바뀌지 않는다. 그래서 비록 이펙트 안에서 ref를 사용하더라도 굳이 의존성 배열에 ref는 추가하지 않아도 된다. 왜냐하면 넣어도 어차피 이펙트 실행을 촉발하지 않을테니까.

## 데이터 패칭시 고려할 점

문서에서도 이펙트 안에서 데이터 가져오기를 **추천하고 있지 않다**. 지금까지 많이 사용해온 패턴임에도 불구하고 뚜렷한 단점이 있어서 TanStack Query, useSWR과 같은 라이브러리를 사용하거나 리액트 풀스택 프레임워크들을 사용하는것을 권장한다.
- **워터폴 현상**을 일으키기 쉽다. 이펙트로 데이터를 가져오면 컴포넌트 트리 위에서부터 부모 컴포넌트가 렌더링되고 데이터를 가져와야, 자식 컴포넌트가 렌더링 된다. 네트워크 환경이 좋지 않다면 전체 과정이 매우 길어지게 된다.
- **캐싱 되지 않는다**. 컴포넌트가 언마운트 되면 가져왔던 데이터가 모조리 사라지고 다시 마운트될 때 데이터를 또 다시 가져와야 한다.
- **코드가 필요 이상으로 길어진다**. 자주 발생하는 버그인 **레이스 컨디션**을 방지하기 위해서 플래그 변수를 넣고, 조건문과 클린업 함수까지 작성하는게 좋은데 이러면 단순한 데이터 패칭 기능에도 많은 줄의 코드가 추가되는 셈.

## 예시 문제

1,2 번 문제는 쉬워서 생략.
### 3. Strict Mode

```jsx

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function onTick() {
      setCount(c => c + 1);
    }

    setInterval(onTick, 1000);
  }, []);

  return <h1>{count}</h1>;
}
```

개발 환경에서는 기본적으로 [Strict Mode](https://react.dev/reference/react/StrictMode#fixing-bugs-found-by-re-running-effects-in-development)가 적용되어 있는데, 이펙트를 두 번 실행한다. 그래서 `setCount(c=>c+1)`이 두 번씩 실행되면서 카운터가 초당 2 씩 증가하게 되는것.

### 4. Race Condition 문제

```jsx
export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    setBio(null);
    fetchBio(person).then(result => {
      setBio(result);
    });
  }, [person]);

  return (
```

이렇게 이펙트에서 **비동기 방식**으로 데이터 패칭을 하면 발생하게 되는 문제가 레이스 컨디션 문제다. `person1` 을 선택했다가 `person2`로 곧바로 바꿨는데, `person1`에 대한 요청의 응답이 더 늦게 도착하면 `person = person2` 인 상태이지만 `bio = person1Bio`가 되는 문제.

```Jsx
useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);
```

그래서 비동기 방식의 데이터 패칭에선  `ignore` 같은 플래그 변수를 사용해야 한다.

