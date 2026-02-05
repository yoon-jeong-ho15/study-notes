---
title: Manipulating the Dom with Refs
date: 2026-01-22
link:
  - https://react.dev/learn/manipulating-the-dom-with-refs
---

### You will learn

- ref로 리액트가 관리하는 DOM 노드에 접근하는 방법
- ref와 `useRef`훅의 관계
- 다른 컴포넌트의 DOM 노드에 접근하는 방법
- 언제 리액트가 관리하는 DOM을 수정해도 되는지

## DOM 노드 조작

기본적으로 리액트는 DOM 조작을 개발자 대신 해준다.
*선언형* 방식으로 UI가 어떻게 그려져야 하는지에 대한 설계도만 작성하면 리액트가 매 순간 그에 알맞게 그려주는것인데, 그래서 특정한 방법을 통해서만 리액트 밖의 요소들을 조종할 수 있다.

공식 문서에서는 **탈출구 "Escape Hatches"** 라고 표현하는데 말 그대로 리액트 밖에 나가게 해주는 방법들이다. 여기에는 ref와 effect 두 방식이 있는데 ref로는 외부에 직접적으로 접근해서 제어할 수 있고, effect로는 리액트 컴포넌트와 외부 시스템을 동기화 할 수 있다. 그리고 둘이 함께 사용되는 경우가 매우 잦다.

```jsx
const myRef = useRef(null);

//...

<div ref={myRef} />
```

이렇게 참조할 노드에 ref를 건내주면 리액트는 `myRef.current`에 이 노드를 넣는다. 그리고 `myRef.current.scrollIntoView()` 처럼 div 노드에 사용할 수 있는 브라우저 API를 사용할 수 있다.

### ref 콜백

사전에 미리 몇개의 ref가 필요한지 알 수 없는 경우도 많을것이다. `items` 의 개수만큼 ref 객체를 만들어야 한다면 어떻게 해야할까?

첫 번째 방법은, `items` 를 담을 부모 노드를 만들고 그 부모 노드에 대한 ref 객체를 만들어 `ref.current.querySelectorAll()`로 자식 노드를 찾는 방법이 있다.

두 번째 방법은 *ref 콜백*을 사용하는 것으로 지정할 노드의 ref 속성에 ref 콜백함수를 넣는방법이다.

```jsx
export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>
	        Neo
        </button>
        <button onClick={() => scrollToCat(catList[5])}>
	        Millie
        </button>
        <button onClick={() => scrollToCat(catList[8])}>
	        Bella
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                map.set(cat, node);

                return () => {
                  map.delete(cat);
                };
              }}
            >
              <img src={cat.imageUrl} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catCount = 10;
  const catList = new Array(catCount)
  for (let i = 0; i < catCount; i++) {
    let imageUrl = '';
    if (i < 5) {
      imageUrl = "https://placecats.com/neo/320/240";
    } else if (i < 8) {
      imageUrl = "https://placecats.com/millie/320/240";
    } else {
      imageUrl = "https://placecats.com/bella/320/240";
    }
    catList[i] = {
      id: i,
      imageUrl,
    };
  }
  return catList;
}
```

위의 코드를 보면 `itemsRef`에 비어있는 map을 지정한다.
이렇게 하면 렌더링이 끝나고 각 `li` 노드별로 ref를 지정할 때 안에 있는 함수가 실행이 되면서 map에 `cat`객체가 키로 각 `li` 노드들이 값으로 들어가게 된다.

즉 두 방법 모두 유동적인 개수대로 ref를 만들어 사용하지 않고 하나의 ref 안에 넣는 방식으로 사용하고 있는데, 이게 가능한 이유는 ref가 **모든 종류의 값**을 담을 수 있는 객체이기 때문이다.

## 다른 컴포넌트의 DOM 노드 접근하기

위의 예시는 한 컴포넌트 안에서 DOM 노드를 조종하기 위해 사용했는데, 다른 컴포넌트에서도 ref를 지정한 노드에 접근할 수 있다.

대표적인 사례가 부모에서 자식 컴포넌트의 DOM 노드에 접근하는 것이다. 부모 컴포넌트에서 비어있는 ref를 생성하고 자식 컴포넌트의 속성으로 넘겨주면, 자식 컴포넌트가 렌더링 될 때 해당 노드에 ref를 지정한다.

> React 19 이전까지는 `forwardRef` 를 사용해야 했으나 이제는 `ref`를 prop으로 전달할 수 있게 됐다.

### `useImperativeHandle`로 접근 범위 제한하기

```Jsx
function MyInput({ ref }) {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input ref={realInputRef} />;
};

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

이렇게 부모에게 자기 노드를 ref로 넘겨주고 있는 `MyInput` 컴포넌트는 부모가 정해진 행동만 할 수 있게 제한할 수 있다. 여기서 정확히는 input 노드의 ref인 `realInputRef`는 자식 컴포넌트 안에 머물고 부모가 접근하는 `inputRef.current`는 자식 컴포넌트에서 `useImperativeHandle`로 만든 특수한 객체다.

이 방법은 자식 컴포넌트가 부모에게 사용 가능한 API를 제공하는것과 같은데, 그래서 API의 이점인 내부 구현 추상화를 얻을 수 있다. 예를들어 자식 컴포넌트에 많은 변화가 있어서 포커스를 줬을때  input창의 색상 변화나 애니메이션이 사용된다고 하자. 그러면 이 변경사항은 부모 컴포넌트의 이벤트 핸들러에 작성되어야 한다. 반면 `useImperativeHandle` 을 사용하면 자식 컴포넌트의 변경사항이 자식 컴포넌트 안에 머무를 수 있다.

## 언제 ref가 연결되는가

리액트에서 화면을 업데이트 하는 과정은 렌더와 커밋 두 단계로 나뉜다.
렌더 단계에서는 컴포넌트들을 읽어 가상 DOM 트리를 만들고, 커밋 단계에서 화면에 적용한다.

그리고 ref는 커밋단계에서 노드들과 연결된다. 이는 즉 ref를 사용하려면 이벤트 핸들러나 이펙트처럼, 화면이 다 그려져야 실행될 수 있는 훅과 함수에서 사용해야 한다는 의미다.

### `flushSync` , 동기 방식으로 상태 업데이트

```Jsx
function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }
```

위의 코드를 보면 새로운 todo를 추가하면 가장 마지막에 추가된 todo, 즉 지금 추가할 todo로 스크롤을 이동하는 이벤트 핸들러라는 것을 알 수 있다.

그런데 여기에는 문제가 있는데, 상태 업데이트가 리액트의 배칭 처리 방식 때문에 '즉시' 수행되는게 아니라는 점이다. `setTodo` 함수가 호출되면 순식간에 새로운 todo 노드가 만들어지긴 하지만 바로 다음에서 `listRef.current` 로 리스트 노드에 접근할때까지는 만들어지지 않는다. 그래서 이전에 만들어진 todo, 한 칸씩 밀려서 스크롤이 이동하게 되는것이다.

```Jsx
  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText('');
      setTodos([ ...todos, newTodo]);
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }
```

이렇게 `flushSync`를 사용하면 상태 업데이트를 '즉시' 진행하고 다음 코드를 읽기 때문에 문제가 해결된다. 이를테면 `await`과 비슷하다고 볼 수 있다.

하지만 리액트를 개발하면서 괜히 리렌더링을 배칭 처리 방식으로 설정한것은 아닐것이다. 따라서 이 배칭 방식을 강제로 무시하는 `flushSync`는 최소한으로 사용되어야 할 것이다.

## 예시 문제

1,2,4번 문제는 쉬워서 생략

### 3. 이미지 스크롤

```jsx
export default function CatFriends() {
  const selectedRef = useRef(null);
  const [index, setIndex] = useState(0);

  return (
    <>
      <nav>
        <button onClick={() => {
          flushSync(() => {
            if (index < catList.length - 1) {
              setIndex(index + 1);
            } else {
              setIndex(0);
            }
          });
          selectedRef.current.scrollIntoView({
            behavior: 'smooth',
            block: 'nearest',
            inline: 'center'
          });
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li
              key={cat.id}
              ref={index === i ?
                selectedRef :
                null
              }
            >
              <img
                className={
                  index === i ?
                    'active'
                    : ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}
```

Next 버튼을 누르면 `index`가 `flushSync`로 업데이트되면서 리렌더링 
-> `index`와 일치하는 노드에 `selectedRef` 연결 
-> `scollIntoView` 