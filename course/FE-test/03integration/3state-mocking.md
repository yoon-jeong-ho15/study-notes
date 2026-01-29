---
title: 상태 관리 모킹하기
date: 2025-10-19
order: 3
---
# 상태 관리 모킹하기

*상태 state* : 시간에 따라 **변할 수 있는 데이터**. 프론트에서 특히 '상태'가 중요하게 다뤄지는 이유는 UI가 상태에 직접적으로 의존하기 때문이다. 

강의 예제에서 `CartTable` 컴포넌트는 다음과 같은 하위 컴포넌트들로 구성되어 있다.

```jsx
const CartTable = () => {
  return (
    <>
      <PageTitle />
      <ProductInfoTable />
      <Divider sx={{ padding: 2 }} />
      <PriceSummary />
    </>
  );
};
```

여기서 `PageTitle`과 `Divider` 컴포넌트는 너무 단순하기 때문에 테스트가 필요하지 않고, 다음 두 개의 컴포넌트는 테스트가 필요하다 :
- `ProductInfoTable` : 장바구니에 담긴 상품 목록
- `PriceSummary` :  장바구니에 담긴 상품들의 가격 총합

위 두 컴포넌트에 대해 각각 통합 테스트를 작성해야 한다. 왜 `CartTable` 컴포넌트에 로직을 응집시켜 **테스트를 하나로 통합하지 않고 굳이 두개로 나눴을까?** 

## zustand 모킹

일단 두 컴포넌트는 모두 zustand 스토어에서 **상태**나 *액션*을 사용한다.그래서 이 상태와 액션을 모킹한다면 두 컴포넌트 모두 테스트할 수 있게 된다.

주로 루트에 `__mocks__` 폴더를 만들고 하위 경로에 모킹파일을 작성해두면 `vitest`나 `jest` 에서 **자동으로 모킹시에 사용**한다. [zustand 문서](https://zustand.docs.pmnd.rs/guides/testing#vitest)

```jsx
// __mocks__/zustand.js
const { create: actualCreate } = await vi.importActual('zustand');
import { act } from '@testing-library/react';

// 앱에 선언된 모든 스토어에 대해 재설정 함수를 저장
const storeResetFns = new Set();

// 스토어를 생성할 때 초기 상태를 가져와 리셋 함수를 생성하고 set에 추가합니다.
export const create = createState => {
  const store = actualCreate(createState);
  const initialState = store.getState();
  storeResetFns.add(() => store.setState(initialState, true));
  return store;
};

// 테스트가 구동되기 전 모든 스토어를 리셋합니다.
// 테스트 독립성 유지
beforeEach(() => {
  act(() => storeResetFns.forEach(resetFn => resetFn()));
});
```

그리고 테스트 환경의 글로벌 설정을 하는 셋업파일(예제에서는 `setupTests.js`)에 다음과 같이 작성한다.

```js
import '@testing-library/jest-dom/vitest'

vi.mock('zustand') // to make it work like Jest (auto-mocking)
```

이렇게 하면 테스트 환경에서 `zustand`를 호출할 때에는 `__mocks__/zustand.js`에 작성된 코드가 호출된다.

### 모킹 유틸 함수

위의 `__mocks__/zustand.js` 파일을 보면 `beforeEach`로 모든 테스트 시작 전에 스토어를 초기화 한다. 
```js
beforeEach(() => {
  act(() => storeResetFns.forEach(resetFn => resetFn()));
});
```

그런데 테스트 하기 전에 특정 데이터가 필요하지 않을까? 예를들어 장바구니에 상품이 있어야 그것의 총액이 잘 계산되는지 검증하거나 장바구니에서 상품을 제거하는 로직도 검증할 수 있지 않을까?

그럴때 사용하는게 **상태를 주입해주는 유틸 함수**다.

```jsx
// src/utils/test/mockZustandStore.jsx
import { useCartStore } from '@/store/cart';
import { useFilterStore } from '@/store/filter';
import { useUserStore } from '@/store/user';

const mockStore = (hook, state) => {
  const initStore = hook.getState();
  hook.setState({ ...initStore, ...state }, true);
};

export const mockUseUserStore = state => {
  mockStore(useUserStore, state);
};

export const mockUseCartStore = state => {
  mockStore(useCartStore, state);
};

export const mockUseFilterStore = state => {
  mockStore(useFilterStore, state);
};

```

(이 모킹 유틸 함수들을 보면 바로 주스탠드 스토어의 **상태를 직접 변경**(`hook.setState()`)하는 것을 볼 수 있다.)

실제 테스트 코드에서 이렇게 호출하여 상태를 설정한다.

```js
mockUseUserStore({ user: { id: 10 } });
```
