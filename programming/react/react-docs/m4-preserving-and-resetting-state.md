---
title: Preserving and Resetting State
date: 2026-02-05
link:
  - https://react.dev/learn/preserving-and-resetting-state
---
### You will learn

- 리액트가 언제 state를 보존하고 언제 초기화 하는지
- 리액트가 강제로 컴포넌트의 state를 초기화 하게 하는 방법
- key값과 타입이 어떻게 state의 보존 여부에 영향을 끼치는지

## state는 렌더 트리 안에 있다

리액트는 컴포넌트 구조를 *렌더 트리* 라는 트리 구조를 만든다. 흔히 컴포넌트 안에서 state를 선언하니까 state가 컴포넌트 안에 위치하고 있다고 생각할 수 있으나 (편의상 그렇게 얘기하기도 하지만), 사실 **state는 컴포넌트가 아닌 리액트에** 있다. 

**컴포넌트는 사실 렌더링 로직**이고, 렌더링이 실행 될 때 필요한 정보들은 리액트가 가지고 있다가 제공하는 것이다. `setState` 함수 호출로 state 업데이트가 발생하면 리액트는 해당 state를 렌더 트리 안에서 찾아 해당 위치의 컴포넌트 렌더링 로직을 새로 업데이트된 state를 주고 실행시킨다. 

## 같은 자리 같은 컴포넌트의 state

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} />
      ) : (
        <Counter isFancy={false} />
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);
  //...
  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
    //...
```

여기서 `score` state의 보존 여부를 생각해보자.

`isFancy` 변화에 따라 `Counter` 컴포넌트가 리렌더링 되지만 state는 보존되는데 왜 그럴까? `Counter` 컴포넌트를 보면 맨 위에 `div` 태그를 가지는데, 리액트가 보는 렌더 트리 안에는 `App`이라는 트리 루트의 첫 자식인 `div`이기 때문이다.

## 같은 위치 다른 컴포넌트의 state

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
```

이렇게 `div`에서 `section` 으로 태그가 완전히 변해버리면 같은 위치라고 하더라도 `score` state는 초기화되어 0으로 돌아간다.


> [!faq] 타입이란? `div` 태그를 가지는 다른 컴포넌트라면?
> 만약 동명의 `score`라는 state를 가지고 맨 위가 `div` 태그로 이루어진 완전히 다른 컴포넌트를 해당 위치에 `Counter`대신 렌더링 한다면 어떻게 될까?
> 같은 위치의  `div`니까  `score` state가 유지될까 아니면 초기화 될까?
>
>**초기화 된다.** 컴포넌트 타입이 HTML 태그의 종류를 의미하는 것은 아니기 때문이다. 문서에서 한 컴포넌트 안에서 다른 컴포넌트를 정의하는 문제에 대해 이야기 한다. 부모 컴포넌트가 렌더링 될 때 마다 **부모 안에 속한 함수인 자식 컴포넌트는 다른 주소값을 가지게 된다.** 그래서 똑같은 자식 컴포넌트로 보여도 주소값이 다르기 때문에 다른 타입이라고 리액트는 판단하고 state를 초기화 한다. 그러니 아예 다른 컴포넌트라면 애초에 다른 주소값을 가지게 되고, 다른 타입이 된다. 

## 같은 위치에서 state 초기화 하기

어떤 이유로 같은 위치에서 같은 컴포넌트를 호출하면서 state가 초기화 되기를 원하는 경우가 있을 수 있다 (문서에서 보여주는 예시는 현실성이 떨어지는것 같아 따로 찾았다) :
- 설문조사와 같은 다단계 폼 - 1번 문항에 답을 하고 다음 단계로 넘어갔을때 1번 문항에 대한 답이 남아있을 수 있다.
- 블로그 에디터나 메모앱 - A글을 수정하고 있다가 저장하지 않고 리스트에서 바로 B글 수정으로 넘어간다면 A글의 내용이 B 글의 본문 위치에 그대로 적용되어 버린다.

이럴때 할 수 있는 효과적인 방법은 컴포넌트에 key값을 주는것이다. key 값은 배열을 렌더링 할 때만 필요한것이 아니다. 리액트는 **key가 변경되면 무조건 다른 컴포넌트라고 판단**한다. 

### 제거된 컴포넌트의 state 보존하기

어떤 이유로는 반대로 사라진 컴포넌트의 state를 기억하고 싶을 수 있다. 위의 예시를 그대로 사용해보면, 1번 문항에 답을 적다가 제출하지 않고 다음 문항으로 넘어갔지만, 다시 돌아왔을때 적었던 답이 남아있기를 원할 수 있다. 이럴땐 어떻게 해야할까?
- 언마운트 대신 CSS로 안보이게 만든다.
- state를 상위로 올리고 각 문항별 답안 state를 부모가 가지고 있다가 자식에게 props로 건내준다.
- 리액트 이외의 다른 저장소를 이용한다 : `localStorage`나 다른 상태 관리 라이브러리들.

## 예시 문제

### 1. 입력값 초기화 문제

```jsx
export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>Hint: Your favorite city?</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>Hide hint</button>
      </div>
    );
  }
  return (
    <div>
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>Show hint</button>
    </div>
  );
}
```

위의 코드에서 `Form` 컴포넌트의 위치가 첫 번째와 두 번째를 오가기 때문에 리렌더링마다 state가 초기화 된다.

해결하기 위해서는 이렇게 항상 컴포넌트의 위치를 고정시켜야 한다.

```jsx
//...
<div>
      {showHint && // p 태그 or null
        <p><i>Hint: Your favorite city?</i></p>
      } 
      <Form />
      {showHint ? (
        <button onClick={() => {
          setShowHint(false);
        }}>Hide hint</button>
      ) : (
        <button onClick={() => {
          setShowHint(true);
        }}>Show hint</button>
      )}
</div>
//...
```

### 2. 입력창 맞바꾸기

```jsx
export default function App() {
//...
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="Last name" /> 
        <Field key="firstName" label="First name" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key="firstName" label="First name" /> 
        <Field key="lastName" label="Last name" />
        {checkbox}
      </>
    );    
  };
}
function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

약간 특수한 경우인데, 리액트가 리렌더링을 할 때 같은 key값을 가진 컴포넌트가 위치만 변경된 경우 state를 보존한다. 

### 5. 리스트에서 state

아래 코드에서 문제는 `Contact` 컴포넌트의 `expanded` 라는 state가 의도와 다르게 남아있게 된다는 점이다. "Alice" 연락처 컴포넌트의 `expanded` 를 참으로 설정하고 위치를 거꾸로 돌리면 해당 위치로 오게되는 "Taylor" 컴포넌트의 `expanded`가 참으로 설정된다. 즉 key값 변경에 따른 상태 초기화가 적용이 되지 않는다는 것.

```jsx
export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Show in reverse order
      </label>
      <ul>
        {displayedContacts.map((contact, i) =>
          <li key={i}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}
```

원인은 key값을 인덱스로 주고 있기 때문에. 배열의 첫 인덱스는 Alice거나 Taylor거나 0인것은 마찬가지다. 그래서 key값이 유지되니 props로 전달되는 `contact` 객체에 대한 정보는 바뀔지라도, 해당 컴포넌트의 state인 `expanded`는 유지가 되는것. 따라서 key값을 배열 인덱스가 아니라 진짜 컴포넌트의 key라고 할 수 있는 값으로(가령 `contact.id`) 올바르게 설정해야 한다.
