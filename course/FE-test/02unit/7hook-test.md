---
title: 리액트 훅 테스트 (act 함수)
date: 2025-10-18
order: 7
---
# 리액트 훅 테스트

강의에서 사용하는 예제의 구조는 이렇다.
`<NavigationBar />`안에 `<ConfirmModal />` 컴포넌트가 있다.
그리고 이 모달을  표시하고 안하고를 `isModalOpened`라는 상태로 결정하는데,
그 상태 관리를 `<NavigationBar />`컴포넌트 안에서 `useState()`훅을 사용해 직접 관리하지 않고,
`useConfirmModal()`이라는 [*커스텀 훅*](https://ko.react.dev/learn/reusing-logic-with-custom-hooks)으로 분리시켜 관리한다.

```jsx
import { useState } from 'react';

const useConfirmModal = (initialValue = false) => {
  const [isModalOpened, setIsModalOpened] = useState(initialValue);

  const toggleIsModalOpened = () => {
    setIsModalOpened(!isModalOpened);
  };

  return {
    toggleIsModalOpened,
    isModalOpened,
  };
};

export default useConfirmModal;
```

이렇게 커스텀 훅으로 분리해 상태를 관리하면 똑같은 기능을 필요로 하는 다른 컴포넌트에서도 사용할 수 있어 코드의 **재사용성**이 높아진다는 장점 외에도 다음과 같은 장점이 있다.
- 관심사 분리로 인한 **가독성** 향상
- 로직 캡슐화로 인한 **테스트 용이성** 증가

## renderHook

테스팅라이브러리에서 제공하는 API.

리액트 훅을 검증하기 위해서는 훅을 호출하고 난 후에 훅이 반환하는 `isMOdalOpened`와 같은 상태가 올바르게 변경되는지 확인해야 한다.
그런데 리액트 규칙상 리액트 훅(리액트 자체 훅, 커스텀 훅 모두)은 **리액트 컴포넌트 안에서만 호출**할 수 있다는 것이다.
지금 테스트를 하고있는 파일인 `useCOnfirmModal.spec.jsx`는 리액트 컴포넌트가 아니기때문에 (일반 js파일) 훅을 호출할 수 없지만, 이를 해결하기 위해 `renderHook()` api를 사용하면 호출할 수 있다.

```jsx
const { result, rerender } = renderHook(useConfirmModal);
// or
const { result } = renderHook(() => useConfirmModal(true));

expect(result.current.isModalOpened).toBe(false);
// or 
```

result : 훅을 호출하여 얻은 결과 값.
rerender : 훅을 원하는 인자와 함께새로 호출하여 상태를 갱신. (강의에서는 보여주기만 하고 사용은 안한다?)

## act 함수

상호 작용(렌더링, 이펙드 등)을 함께 그룹화하고 실행해 렌더링과 업데이트가 **실제 앱이 동작하는 것과 유사한 방식**으로 동작함.
- 즉 act를 사용해야 테스트 환경의 **가상의 돔(jsdom)에 제대로 반영**된다.
컴포넌트를 렌더링한 뒤 업데이트 하는 코드의 결과를 검증하고 싶을때 사용.

> 그런데 왜 이전 단위 테스트(UI 컴포넌트)에서는 act함수가 없이 테스트를 수행할 수 있었을까?
> 왜냐하면, render 함수 자체에서 내부적으로 act 함수를 사용하기 때문.

```jsx
// 실패
it('훅의 toggleIsModalOpened()를 호출하면 isModalOpened 상태가 toggle된다.', () => {
  const { result } = renderHook(useConfirmModal);
  result.current.toggleIsModalOpened();
  expect(result.current.isModalOpened).toBe(true);
});

//성공
it('훅의 toggleIsModalOpened()를 호출하면 isModalOpened 상태가 toggle된다.', () => {
  const { result } = renderHook(useConfirmModal);
  act(() => {
    result.current.toggleIsModalOpened();
  });
  expect(result.current.isModalOpened).toBe(true);
});
```
