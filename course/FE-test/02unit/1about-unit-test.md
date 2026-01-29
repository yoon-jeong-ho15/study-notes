---
title: 유닛(단위) 테스트란
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

**Arrange** : 테스트를 위한 환경 만들기 (컴포넌트 렌더링 하는 코드 등)
- testing-library 사용
**Act** : 테스트할 동작 발생
**Assert** : 올바른 동작이 실행되었는지 또는 변경사항 검증
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
