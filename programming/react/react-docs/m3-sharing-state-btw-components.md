---
title: Sharing State Between Components
date: 2026-02-05
link:
  - https://react.dev/learn/sharing-state-between-components
---
### You will learn

- 'state 끌어 올리기'로 어떻게 컴포넌트간에 state를 공유할 수 있는지
- 제어 컴포넌트, 비제어 컴포넌트란 무엇인지

## Lifting state up

두 컴포넌트의 state가 동시에 업데이트되기를 원할 수 있다. 그러기 위해서는 각 컴포넌트에서 state를 제거하고 가장 가까운 둘의 공통 부모 컴포넌트에서 state를 선언한다. 그리고 부모가 하위 컴포넌트에게 전달한다. 이 방법을 **state 끌어올리기** 라고 한다.

- `Accordion`
    - `Panel`
    - `Panel`

여기서  하나의 `Panel` 컴포넌트만 열리길 원한다면? 즉 한 `Panel` 컴포넌트를 열면 다른 `Panel` 컴포넌트를 닫고 싶다면? 각 `Panel` 컴포넌트에서 **다른 컴포넌트의 state를 업데이트할 수 없기 때문에** 둘의 부모인 `Accordion` 에서 어떤 `Panel` 컴포넌트를 보여줄지 결정하는 state를 선언하고 자식에게 건내줘야 한다. 즉 state 끌어올리기가 필요하다.

```jsx
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        //...
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        //...
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Show
        </button>
      )}
    </section>
  );
}
```

이렇게 자식에게 두 가지를 전달한다. 
1) **state** 와 
2) state 를 업데이트하는 **이벤트 핸들러**

### 제어 controlled vs 비제어 uncontrolled 

바로 위의 방식처럼 부모 컴포넌트에서 **중요한 정보를 props로 전달**하는 경우 자식 컴포넌트들을 제어 컴포넌트라고 한다. 이렇게 하면 자식이 어떻게 동작할지 부모가 결정할 수 있기 때문이다.

반대로 자식 컴포넌트가 중요한 정보를 독립적으로 **지역 state를 갖는** 경우 비제어 컴포넌트라고 한다. 비제어 컴포넌트는 코드를 작성할 때 간편할 수 있지만, 여러 컴포넌트를 조합하여 사용할 때 유연하지 않을 수 있다.

많은 경우 "제어"와 "비제어"가 분명히 구분되지 않는다. 하지만 이 구분을 통해 어떤 정보가 props로 제어되어야 하는지 어떤 정보는 state로 비제어 되어야 하는지 생각해보는 관점을 제공한다.

## 예시 문제

### 2. 검색어 필터링

입력창에 넣은 검색를 반영하여 필터링 된 결과만 보여주도록 수정하는 문제. 

```jsx
export default function FilterableList() {
  const [query, setQuery] = useState('');
  return (
    <>
      <SearchBar query={query} onChange={(e)=>setQuery(e.target.value)} />
      <hr />
      <List items={foods} query={query} />
    </>
  );
}

function SearchBar({query, onChange}) {
  return (
    <label>
      Search:{' '}
      <input
        value={query}
        onChange={onChange}
      />
    </label>
  );
}

function List({ items,query }) {
  const result = filterItems(items,query);
  return (
    <table>
      <tbody>
        {result.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

나는 위에처럼 `List` 컴포넌트에서 필터링을 진행했는데 답안에서는 부모 컴포넌트인 `FilterableList`에서 필터링한 후에 `List` 컴포넌트에 필터링 결과를 건내주도록 설계했다.

부모에서 계산하고 결과를 건내주는 답안의 방식이 더 권장되는 방법이다. 
- `List` 컴포넌트가 **특정 로직에 종속되지 않게** 되고 (재사용성)
- `List` 컴포넌트는 **UI 컴포넌트**로서 하나의 책임만 갖는다. (관심사 분리)
