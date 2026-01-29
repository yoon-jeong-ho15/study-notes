---
title: 타이머 테스트
date: 2025-10-18
order: 8
---
# 타이머 테스트

*디바운스* : 특정 함수의 호출 횟수를 제한
- 주로 스크롤처럼 대량의 이벤트 핸들러가 발생할 때 성능 개선을 목적으로 사용
```js
export const debounce = (fn, wait) => {
  let timeout = null;

  return (...args) => {
    const later = () => {
      timeout = -1;
      fn(...args);
    };

    if (timeout) {
      clearTimeout(timeout);
    }
    timeout = window.setTimeout(later, wait);
  };
};
```

## 타이머 모킹

```jsx
beforeEach(() => {
    vi.useFakeTimers();
});
afterEach(() => {
	vi.useRealTimers();
});
  
it('연이어 호출해도 마지막 호출 기준으로 지정된 타이머 시간이 지난 경우에만 함수가 호출된다.', () => {
    const spy = vi.fn();

    const debouncedFn = debounce(spy, 300);

    debouncedFn();
    vi.advanceTimersByTime(200);
    debouncedFn();
    vi.advanceTimersByTime(100);
    debouncedFn();
    vi.advanceTimersByTime(200);
    debouncedFn();
    vi.advanceTimersByTime(300);
    debouncedFn();

    expect(spy).toHaveBeenCalledTimes(1);
});
```
