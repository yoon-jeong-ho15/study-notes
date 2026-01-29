---
title: userEvent를 사용한 사용자 상호작용 테스트
date: 2025-10-18
order: 9
---
# userEvent와 사용자 상호작용

*fireEvent* : userEvent처럼 리액트 테스팅 라이브러리에서 DOM 이벤트를 시뮬레이션 하기 위해 제공하는 api

## userEvent vs fireEvent ?

fireEvent는 "**해당 이벤트만 디스패치**" 한다.

이게 무슨뜻이냐면, `userEvent`로 클릭 이벤트를 발생시킨다면 mouseOver -> mouseMove -> mouseDown -> mouseUp -> click 의 순서대로 **실제 사용자가 사용할 때 처럼** 여러 이벤트가 발생하지만,
`fireEvent`는 단순히 click 이벤트만 발생시킨다는 것.

이전 시간에 텍스트 인풋을 클릭하면 onFocus가 호출되는지 테스트를 했는데, 이 때 사용한것은 `userEvent`.
만약 `fireEvent`로 텍스트 인풋을 클릭한다면 onFocus가 호출되지 않는다.

그래서 되도록 userEvent 사용이 불가능한 경우가 아니라면 userEvent를 사용한다.
- 불가능한 경우 : ex.scroll