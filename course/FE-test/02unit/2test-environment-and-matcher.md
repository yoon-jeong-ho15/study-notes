---
title: 테스트 환경과 매처
date: 2025-10-18
order: 2
---
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