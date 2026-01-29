---
title: React Testing Library와 컴포넌트 테스트
date: 2025-10-18
order: 4
---
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

