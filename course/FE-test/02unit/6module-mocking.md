---
title: 모듈 모킹
date: 2025-10-18
order: 6
---
# 모듈 모킹

`shopping-mall-unit-test` 브랜치에서 `EmptyNotice`와 `NotFoundPage`,`ErrorPage` 컴포넌트들을 단위 테스트 한다.
- `EmptyNotice` : 장바구니가 비어있는 상태에서 장바구니 페이지로 이동시에 장바구니가 비어있다고 보여주는 컴포넌트
	- 이 컴포넌트는 사실 **여러 컴포넌트들이 조합되어 이루어진 컴포넌트임에도 불구**하고 홈으로 이동한다는 **하나의 기능**만 가지고 있기 때문에 독립적인 컴포넌트라고 볼 수 있다.
- `NotFoundPage` : 사용자가 잘못된 경로로 이동했을 때 보여주는 페이지
- `ErrorPage` : 예상하지 못한 오류가 발생했을 때 보여주는 페이지
	- 두 에러 페이지 모두 **페이지 단위의 큰 컴포넌트임에도 불구**하고 굉장히 **단순한 구성 요소**를 가지고있고 마찬가지로 '뒤로 이동'과 같은 **간단한 기능**만 가지고 있기 때문에 단위 테스트로 검증할 수 있다.

그런데 위 세 컴포넌트 모두 `react-router-dom`의 `useNavigate()`훅을 가져와 사용하고있다. (즉 `react-router-dom`에 **의존성**을 가지고 있다.)
이 컴포넌트들을 단위 테스트 할때는 해당 기능을 **제대로 호출하는지만 검증**하면 된다. (외부 모듈의 검증은 해당 모듈 자체에서 테스트를 통해 검증할 일이지 참조하고 있는 여기서 할 일은 아님)
그리고 이 외부 모듈의 특정 기능을 제대로 호출하는지 알아보기 위해 **모킹**이라는 것을 한다.

*모킹* : 실제 모듈,객체와 동일한 동작을 하도록 만든 모의 모듈,객체로 실제를 대체하는 것.
- 모킹을 통해 우리는 외부 모듈과 검증할 모듈을 분리해서 필요한 검증만 진행할 수 있다.
- 그러나 실제 모듈과 완전히 동일한 모의 객체를 구현하는 것은 큰 비용이 드는 작업이며, **(정확히 무슨뜻일까?)**
- 모의 객체를 남용하는 것은 테스트 신뢰성을 낮추는 결과를 낳는다.

```jsx
const navigateFn = vi.fn();

vi.mock('react-router-dom', async () => {
  const original = await vi.importActual('react-router-dom');

  return { ...original, useNavigate: () => navigateFn };
});

it('Home으로 이동 버튼 클릭시 홈 경로로 이동하는 navigate가 실행된다', async () => {
  const { user } = await render(<NotFoundPage />);

  const button = await screen.getByRole('button', { name: 'Home으로 이동' });

  await user.click(button);

  expect(navigateFn).toHaveBeenNthCalledWith(1, '/', { replace: true });
});
```

`vi.mock()`함수를 통해 `react-router-dom`의 모킹을 만든다.
그러면 `<NotFoundPage />` 컴포넌트가 불러오는 해당 모듈은 진짜 모듈이 아닌 여기서 만들어진 모킹이 된다.
그 컴포넌트에서 `useNavigate()`를 호출할 다음과 같이 호출할 때
```jsx
  const handleClickNavigateHomeButton = () => {
    navigate(pageRoutes.main, { replace: true });
  };
  
  return(
  ...
  <button onClick={handleClickNavigateHomeButton}>Home으로 이동</button>
  )
```
 실제로 호출되는 것은 모킹에 넣었던 `navigateFn` 스파이 함수다.
 그래서 `expect()`매처로 `useNavigate()`함수를 호출하면서 어떤 인자를 넣었는지 (`"/", {replace : true}`)확인할 수 있게 된다.

## 모킹 초기화
위에서 `useNavigate()`와 `react-router-dom` 모듈을 모킹했다.
그런데 이 모킹이 다른 테스트에 영향을 줄 수 있는 경우(다른 테스트에서는 다른 방식으로 모킹을 해야한다던가)를 위해 테스트 수행 후에는 모킹을 초기화하는것이 좋다.

```js
//setupTests.js
...

beforeAll(() => {
  server.listen();
});

afterEach(() => {
  server.resetHandlers();
  vi.clearAllMocks();
});

afterAll(() => {
  vi.resetAllMocks();
  server.close();
});

...
```

`clearAllMocks()` :
- 모킹된 모의 객체 호출에 대한 히스토리를 초기화
	- 히스토리가 초기화되지 않고 계속 쌓이면 **호출 횟수**나 **인자**가 변경되어 다른 테스트에 영향을 줄 수 있다.
- 모킹된 모듈의 구현을 초기화 하지는 않는다 (모킹된 상태로 유지)

`resetAllMocks()` :
- 모킹된 모듈의 구현을 초기화
