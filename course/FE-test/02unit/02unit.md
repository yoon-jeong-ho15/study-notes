---
title: 단위 테스트
date: 2025-10-17
order: 1
---
# '단위 테스트'란?

앱에서 테스트 가능한 **가장 작은 소프트웨어**를 실행해 예상대로 동작하는지 확인하는 테스트.
- 프론트에서는 가령 **단일**함수의 결과값, **단일** 컴포넌트의 상태나 행위 등이 될 수 있다.
- '단일' : 컴포넌트끼리의 상호작용을 검증하기보다는 각각의 행위를 독립적으로 검증한다.

프론트(리액트)에서는 버튼, 텍스트 인풋, 캐러셀, 아코디언 등의 컴포넌트를 재사용하는 경우가 다반사인데 이런 컴포넌트들이 단위 테스트에 아주 적합한 컴포넌트이다. (Atomic 컴포넌트라고 부르기도 한다)

## AAA 패턴

테스트 코드 작성에도 여러 패턴이 있다. AAA 패턴이란 Arrange-Act-Assert 패턴의 줄임말.

Arrange : 테스트를 위한 환경 만들기 (컴포넌트 렌더링 하는 코드 등)
- testing-library 사용
Act : 테스트할 동작 발생
Assert : 올바른 동작이 실행되었는지 또는 변경사항 검증
- vitest 라이브러리 사용

```jsx
it('className prop으로 설정한 css class가 적용된다.', async () => {
  // Arrange
  // - className을 지닌 컴포넌트 랜더링
  await render(<TextField className={`my-class`} />);
  
  // Act
  // - 클릭이나 메소드 호출, prop 변경 등등에 대한 작업
  
  // Assert
  // - 렌더링 후 DOM에 해당 class가 존재하는지 검증
  expect(screen.getByPlaceholderText('텍스트를 입력해 주세요.')).toHaveClass(
    'my-class',
  );
});
```

# 테스트 환경과 매처

*테스트 프레임워크* :  태스크 러너, 어설션 라이브러리, 플러그인 등 테스트 실행 및 검증에 필요한 환경을 제공하는 도구 (Vitest, Cypress 등)

## Vitest 설정

`vite.config.js`파일의 `test` 필드에 정의하거나, 코드량이 많다면 `vitest.config.js`라는 파일을 만들기도 한다.
```js
//vite.config.js
export default defineConfig({
  plugins: [react(), eslint({ exclude: ['/virtual:/**', 'node_modules/**'] })],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/utils/test/setupTests.js',
  },
  resolve: {
    alias: [{ find: '@', replacement: path.resolve(__dirname, 'src') }],
  },
});
```

## JSDOM
테스트는 Node.js 환경에서 실행되는데, 브라우저 없이 HTML의 구조를 확인할 수 있게 돕는 라이브러리.
`screen.debug()`함수를 사용하면 다음과 같이 렌더링 결과물을 HTML 태그들로 나타내준다.
```html
<body>
  <div>
    <input
      class="text-input my-class"
      placeholder="텍스트를 입력해 주세요."
      type="text"
      value=""
    />
  </div>
</body>
```

## `it()` 함수

테스트가 통과하기 위한 조건을 기술하여 검증을 실행한다.
it 함수는 간단히 test 함수의 alias. test 함수와의 기능상의 차이는 없다.

### describe()
기본적으로 it함수로 정의된 테스트 파일은 '루트'이기 때문에 최상위 컨텍스트에서 테스트가 실행된다.
describe 함수를 사용하면 여러 it 함수들을 그룹지을 수 있다.

### 매처 Matcher
기대 결과를 검증하기 위해 사용되는 API 집합. (위의 `expect().toHaveClass()`)

# setup 과 teardown

테스트의 원칙중 '모든 테스트는 **독립적으로 실행**되어야 한다'는 원칙이 있다.

가령 테스트 A와 B가 있다고 하고, 테스트 A를 실행하고 B를 실행하면 성공하지만 순서를 바꿔 B를 먼저 실행하면 실패하는 경우가 있다. 이런 경우 이 테스트 코드는 잘못 작성된것이다. (전역변수를 사용해 조건 처리를 했을 수 있다)
테스트들은 순서, 다른 테스트의 실행 여부와 상관 없이 완전히 독립적으로 수행되어야 한다.

이런 독립성을 보장하기 위해 setup과 teardown이 사용된다.
setup : 테스트를 실행하기 전 수행해야 하는 작업.
	- beforeEach : 테스트 파일 혹은 스코프(describe함수) 내에 모든 테스트가 실행되기 전에 호출
	- beforeAll : 파일 혹은 스코프 내의 테스트가 실행되기 전 **단 한번**만 호출
teardown : 테스트를 실행한 뒤 수행해야 하는 작업.
	- afterEach 
	- afterAll 

# React Testing Library와 컴포넌트 테스트

특정 개발(리액트 외 뷰,스벨트에서도), 테스트 프레임워크에 종속되지는 않는다.

Testing Library의 핵심 철학은 UI 컴포넌트를 **사용자가 사용하는 방식**으로 테스트 하는것.
- 사용자가 사용하는 방식 : DOM 노드 조회, 사용자와 비슷한 방식으로 이벤트 발생.
- 테스트 코드 작성 원칙중 인터페이스를 기준으로 작성해야한다는 원칙과 일치.
- getBy api중 role, label-text, image-alt-text 처럼 사용자 관점에 가까운 api를 사용한다.

## userEvent

```jsx
// TextField.spec.jsx
import render from '@/utils/test/render';

it('텍스트를 입력하면 onChange prop으로 등록한 함수가 호출된다', async () => {
  const { user } = await render(<TextField />);
  const textInput = screen.getByPlaceholderText('상품명을 입력해 주세요.');
  await user.type(textInput, 'test');
  ...
});

// render.jsx
import { render } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

export default async component => {
  const user = userEvent.setup();

  return {
    user,
    ...render(component),
  };
};
```

`userEvent.setup()` 함수가 반환하는 `user`객체는 사용자의 행동을 시뮬레이션 하는 이벤트를 발생시킬 수 있다.

## spy 함수

*`vi.fn()`* 함수를 호출해 만들 수 있다.
스파이 함수는 테스트 코드에서 특정 함수가 **호출되었는지**, 함수의 **인자로 어떤 것**이 넘어왔고 **어떤 값을 반환**하는지 등 다양한 값들을 저장하고 있다.
컴포넌트 안에서 실행되는 함수를 검증하기 위해 필수다.
```jsx
it('텍스트를 입력하면 onChange prop으로 등록한 함수가 호출된다', async () => {
  const spy = vi.fn();

  const { user } = await render(<TextField onChange={spy} />);
  const textInput = screen.getByPlaceholderText('텍스트를 입력해 주세요.');
  await user.type(textInput, 'test');

  expect(spy).toHaveBeenCalledWith('test');
});
```

