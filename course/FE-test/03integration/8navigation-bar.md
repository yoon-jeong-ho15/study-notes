---
title: 통합 테스트 작성 - NavigationBar
date: 2026-01-29
order: 8
---
# NavigationBar 컴포넌트

`NavigationBar`컴포넌트의 기능들을 정리하면 다음과 같다
- 로고 클릭시 메인 페이지로 이동
- 비로그인시 
	- 로그인 버튼 노출, 클릭시 로그인 페이지로 이동
- 로그인시 
	- 장바구니 버튼
		- 장바구니에 담긴 상품수가 나타남
		- 클릭시 장바구니 페이지로 이동
	- 로그아웃 버튼
		- 클릭시 "로그아웃 하시겠습니까?" 모달 노출
		- 모달의 확인 버튼 클릭시 로그아웃

이제 통합 테스트를 어떻게 작성하면 좋을까? 예제에서는 이렇게 구성했다.
- (it)로고 클릭시 "/" 경로로 이동(`navigate`를 호출한다).
- (describe)로그인이 된 경우
	- (it)장바구니 버튼, 장바구니에 담긴 상품 수, 로그아웃 버튼(사용자 이름)이 렌더링 된다
	- (it)장바구니 버튼 클릭시 "/cart" 경로로 이동한다(`navigate`를 호출한다).
	- (describe)로그아웃 버튼
		- (it)모달이 "로그아웃 하시겠습니까?" 문구와 함께 렌더링 된다.
		- (it)확인 버튼을 누르면 로그아웃이 되며 모달이 사라진다
		- (it)취소 버튼을 누르면 모달이 사라진다.
- (describe)비로그인 경우
	- (it)로그인 버튼 렌더링, 클릭시 현재 "/login"경로로 `pathname`과 함께 `navigate` 함수를 호출

## msw의 `use`함수

로그인이 된 경우 테스트를 하기 위해서는 로그인이 된 상태를 모킹해야 한다.
그러려면 로그인이 된 경우의 describe 안에 셋업 `beforeEach`로 해줘야한다.
그런데 문제는 이러한 사용자 정보는 **스토어에서 가져오는것이 아니라 api 호출을 통해 가져온다**는 것이다. 

`NavigationBar`컴포넌트에서는 `useProfile`훅(내부적으로 리액트 쿼리를 사용)을 사용해 데이터를 받아와 `setUserData`로 스토어에 저장한다.

```js
//NavigationBar.jsx
const { data, remove } = useProfile({
    config: {
      onSuccess: profile => {
        setUserData(profile);
        initCart(profile.id);
      },
      enabled: !!isLogin,
    },
  });
```

그래서 이 api를 모킹해야하는데, 저번 시간부터 우리는 api 모킹을 msw로 해왔다. 그래서 msw 코드(`handlers.js`) 로 가봐야 하는데, `useProfile`훅이 요청을 보내는 주소에 해당하는 코드를 찾아보면 이렇다.

```js
  rest.get(`${API_DOMAIN}${apiRoutes.profile}`, (req, res, ctx) => {
    return res(ctx.status(200), ctx.json(null));
  }),
```

로그인이 되어있지 않는게 기본값의 상태라고 보는게 적절하니까 위의 코드를 변경할 수는 없다. 그래서 사용하는게 msw의 `use` 함수다.

### server 인스턴스

테스트 셋업 파일에 가보면 `server` 인스턴스를 사용하고 있는것을 볼 수 있다.

```js
//setupTests.js
import { setupServer } from 'msw/node';
import '@testing-library/jest-dom';

import { handlers } from '@/__mocks__/handlers';

/* msw */
export const server = setupServer(...handlers);

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

//...

```

이 인스턴스를 그대로 사용해야 **기존에 모킹된 api의 응답을 정상적으로 변경**할 수 있다.

### `use`함수 작성

```js
describe('로그인이 된 경우', () => {
  const userId = 10;

  beforeEach(() => {
    server.use(
      rest.get('/user', (_, res, ctx) => {
        return res(
          ctx.status(200),
          ctx.json({
            email: 'maria@mail.com',
            id: userId,
            name: 'Maria',
            password: '12345',
          }),
        );
      }),
    );
    mockUseUserStore({ isLogin: true });
```

`use`함수를 사용하면 `handlers.js`에 작성된 기본 설정대로가 아니라 `use`함수 내부에 작성한 응답을 기준으로 테스트가 진행될 수 있다. 이 환경에서 `useProfile`훅은 위에 작성된 객체를 반환한다.

### `use`함수 결과 초기화

`use`함수로 변경한 서버의 api 응답도 다른 모킹들과 마찬가지로 초기화를 해주어야 하는데, 어디서 해주는것일까?
위의 `setupTests.js`를 보면 다음과 같은 코드에서 모든 테스트가 완료된 후에 실행하는 티어다운으로 작성되어 있는것을 볼 수 있다.

```js
//setupTests.js
afterEach(() => {
  server.resetHandlers();
  vi.clearAllMocks();
});
```

## `within` 함수

로그아웃 버튼을 누르면 모달이 나타나는지에 대한 테스트코드다.

```jsx
    it('모달이 렌더링되며, 모달 내에 "로그아웃 하시겠습니까?" 텍스트가 렌더링된다.', () => {
      const dialog = screen.getByRole('dialog');

      expect(screen.getByRole('dialog')).toBeInTheDocument();
      expect(
        within(dialog).getByText('로그아웃 하시겠습니까?'),
      ).toBeInTheDocument();
    });
```

여기 코드를 보면 `screen.getByText` 대신 `within(dialog).getByText`를 쓰고있다.
`within` 함수를 쓰면 `getBy`의 조회 범위를 해당 요소로 제한할 수 있다. 여기서는 사실 "로그아웃 하시겠습니까?"라는 문구가 페이지 전체에 하나밖에 없어서 문제가 되지 않지만, 가령 `ProductList` 컴포넌트를 테스트할 때 여러 `ProductCard`가 화면상에 존재하고 거기서 **특정 상품의 "장바구니" 버튼**을 찾으려고 한다면 화면상에 여러 "장바구니" 버튼이 존재하기 때문에 `screen` 대신 `within`을 반드시 사용해야 한다.

