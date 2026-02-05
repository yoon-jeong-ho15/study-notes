---
title: Choosing the State Structure
date: 2026-01-30
link:
  - https://react.dev/learn/choosing-the-state-structure
---
### You will learn

- 언제 하나의 혹은 여러개의 state 변수를 사용해야 하는지 
- state를 구성할 때 피해야 하는 것들
- 흔한 state 구조의 문제들은 무엇이고 어떻게 고치는지

## state 구조화의 원칙

state 구조화의 완성도는 수정하고 디버깅하기 즐거운 컴포넌트와 괴로운 컴포넌트를 결정한다고 한다.

1. **관련있는 state 그룹화** - 두 개 이상의 state 변수가 언제나 함께 업데이트 된다면 이 변수를 단일 state 객체로 합친다.
2. 모순된 state 피하기 - 두 개가 동시에 `true`일 수 없는 state들처럼 서로 모순을 일으키는 state가 있으면 수정한다.
3. **불필요한 state 만들지 않기** -  여기서 "불필요"하다는 말은 다른 state나 props에서 계산될 수 있는 정보라면 state로 해당 정보를 관리 할 필요가 없다는 뜻이다.
4. state 중복 피하기 - 같은 정보가 여러 state에 중복되어 존재할 경우 **동기화하기 어렵기 때문에** 피해야 한다.
5. **깊게 중첩된(nested) state 피하기** - 깊은 구조를 가진 객체 형태의 state는 업데이트하기 쉽지 않다. 그래서 가능한 한 평탄한(flat) 구조를 가지도록 설계한다.
(하이라이트 하지 않은 원칙은 이전 문서에서 다룬 중복된 내용임)

### 1. state 그룹화

언제 여러 정보를 하나의 state 변수로 사용해야하고 언제는 각각 따로 여러개의 state 변수를 사용해야 할까? 마우스 좌표처럼 거의 항상 동시에 업데이트되는 정보들인 경우 별도의 state로 두는것보다 하나로 병합시키는것이 좋다.

```jsx
// X
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// O
const [position, setPosition] = useState({ x: 0, y: 0 });

return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
    //...
```

#### state 객체의 일부 필드만 업데이트 할 수 없다.

```jsx
setPosition({x:100});
```

이렇게 `position` 의 `x`필드만 업데이트 할 수 없다. 항상 기존 `position`의 모든 필드를 업데이트 해줘야 한다. 

```jsx
setPosition({...position, x:100});
```

이렇게 하면 `x` 필드를 제외한 모든 값들을 그대로 사용하면서 `x`의 값만 바꿔서 새로운 state로 업데이트할 수 있다.

### 2. 모순되는 state

```jsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }
  
  //...
```

위의 코드는 정상적으로 작동하지만  `isSending`과 `isSent`는 모순되는 상황을 초래할 수 있다. 예를들어 `setIsSending(false)`를 했지만 `setIsSent(true)` 해주는것을 깜빡할 수도 있고, `isSending`과 `isSent`가 동시에 참이되는 역설을 만들어낼 수 있다. 

그래서 `status`로 state를 통합하고 `sending`, `sent`, `typing`(초기값) 중에 하나의 값을 가지도록 하면 된다.

#### 상수 선언

가독성을 위해서(`isSending`혹은 `isSent`가 확실히 읽기 편하다) 이렇게 상수로 선언할 수 있다. 

```jsx
const isSending = status === 'sending';
const isSent = status === 'sent';
```

### 3. 불필요한 state

```jsx
export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }
```

여기서 `fullName` 는 굳이 state 변수로 관리 할 필요가 없다. `firstName`과 `lastName`을 통해 계산될 수 있는 값이기 때문이다.

```jsx
const fullName = firstName + ' ' + lastName;
```

이렇게 일반 변수로 선언하여 관리하면, 렌더링마다 최신 값을 읽을 수 있으면서도 굳이 `setFullName`으로 업데이트를 할 필요도 없어진다.

### 4. 중복된 state

```jsx
export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );
```

위의 코드를 보면 `selectedItem` state에 item 객체로 저장하고 있다. 하지만 이건 좋은 패턴이 아니다. `items`에 이미 있는 정보를 그대로 `seletedItem`도 가지고 있기 때문이다. 이 때 발생할 수 있는 문제는 두 정보 사이의 동기화가 문제가 될 수 있다.

만약에 이렇게 각 item의 이름을 변경할 수 있다고 해보자 :

```jsx
function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }
  
  return(
  //...
	<ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
```

여기서 아이템 이름을 변경한다고 하면 `items`에 있는 해당 아이템만 변경되고 `selectedItem`은 변경되지 않는다. 
"pretzels" 라는 아이템을 선택했다면`selectedItem === {id:0, name:"pretzels"}` 이렇게 된다. 그런데 이때 "pretzels"의 이름을 "Pretzels 프레첼"로 변경한다면 

```js
items = {
  {
    id : 0,
    name : "Pretzels 프레첼"
  }
  //...
}
```

이렇게 되지만 `selectedItem`은 그대로 `{id:0, name:"pretzels"}` 로 남아있게 된다. (`selectedItem`도 함께 바꿔주면 되는 간단한 문제지만 항상 깜빡하지 않는 것은 쉽지 않은 일이고, 애초에 문제의 가능성을 차단하는게 최선이다.)

그래서 이렇게 `selectedId` 를 state로 가져가고 `selectedItem`은 일반 변수로 선언해야 한다.

```jsx
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find(item =>
    item.id === selectedId
  );
```

### 5. 깊게 중첩된 state

다음과 같은 구조를 가진 state를 가정해보자

```js
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, 
      //....
      ]
    }, {
      id: 10,
      title: 'Americas',
      childPlaces: [{
        id: 11,
        title: 'Argentina',
        childPlaces: []
      },  
      //...
      ]
    }, {
      id: 19,
      title: 'Asia',
      childPlaces: [{
        id: 20,
        title: 'China',
        childPlaces: []
      },
      //...
      ]
    }, {
      id: 26,
      title: 'Europe',
      childPlaces: [{
        id: 27,
        title: 'Croatia',
        childPlaces: [],
      }, 
      //...
      ]
    }, {
      id: 34,
      title: 'Oceania',
      childPlaces: [{
        id: 35,
        title: 'Australia',
        childPlaces: [],
      },
      //...
      ]
    }]
  }, {
    id: 42,
    title: 'Moon',
    childPlaces: [{
      id: 43,
      title: 'Rheita',
      childPlaces: []
    }, {
      id: 44,
      title: 'Piccolomini',
      childPlaces: []
    }, {
      id: 45,
      title: 'Tycho',
      childPlaces: []
    }]
  },
  //...
};
```

이 구조에서 장소를 추가하거나 삭제하는 코드는 매우 복잡하고 길어질 것이다. (state업데이트는 일부 값만 변경할 수 없고 전체 값을 업데이트 해야한다. 예시 - `setState({...state, newState})`)

하지만 당연히도 이렇게 큰 state가 필요할 수가 있다. 이렇게 큰 state를 만들면 안되는것은 아니다. 이때 적용 가능한 팁이 "**평탄화**"다.

```js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  //...
}
```

이렇게 하면 `initialTravelPlan` 은 깊은 계층을 가지는 복잡하게 꼬여있는 객체가 아니라 하나의 계층으로 이루어진 단순한 맵 형태가 된다. **마치 데이터베이스 테이블 처럼**. 

이제 평탄하게 만든(**정규화 normalize** 라고도 부른다.) state를 업데이트 하는것은 쉬워진다.

```jsx
// 지우는 로직
function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Update the root state object...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      [parentId]: nextParent
    });
  }
```

#### 재귀 렌더링

이렇게 **트리 구조**(나쁜 예시에서는 진짜 트리 구조로 되어있었고, 정규화 된 후에는 개념적으로 트리 구조를 가지고 있다)컴포넌트를 재귀 호출하는 방식(DFS)으로 렌더링 할 수 있다. 

```jsx
function PlaceTree({ id, placesById }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              placesById={placesById}
            />
          ))}
        </ol>
      )}
    </li>
  );
}
```

## 예시 문제

### 1. 업데이트되지 않는 컴포넌트

```jsx
export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```

props로 `color`와 `time`을 받고있는데 또 다시 `color` state를 선언하여 가진다. 상위 컴포넌트에서 색을 변경해도 state에 영향을 주지 않게된다(컴포넌트가 마운트될 때 가졌던 초기값을 state로 간직하지만, 그걸 변경하는 트리거가 없기 때문에). state를 무분별하게 사용하여 발생하는 문제. 직접 `props.color`를 받아 사용하면 부모의 state 변경에 맞춰 색이 변하게 된다.

### 2. 고장난 체크리스트

물품을 체크한 후에 삭제를 해도 카운터에 반영되지 않는 등의 문제가 있다.

```jsx
let nextId = 3;
const initialItems = [
  { id: 0, title: 'Warm socks', packed: true },
  { id: 1, title: 'Travel journal', packed: false },
  { id: 2, title: 'Watercolors', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);
  const [total, setTotal] = useState(3);
  const [packed, setPacked] = useState(1);

  function handleAddItem(title) {
    setTotal(total + 1);
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    if (nextItem.packed) {
      setPacked(packed + 1);
    } else {
      setPacked(packed - 1);
    }
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setTotal(total - 1);
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>  
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

`total`, `packed`같은 불필요한 state들을 가지기 떄문에 발생하는 문제. 정확히는 이 state들을 제대로 변경하지 않기 때문에 발생하는 문제이지만. 
- `handleDeleteItem`에서 `packed` state를 업데이트 하지 않는다.

### 3. 선택 사라짐 

코드가 '정상적'으로 작동을 하기는 한다. 다만 "star"와 "unstar"를 눌러 둘 사이를 오갈 때 잠깐 호버할때 보이는 하이라이트 효과가 잠깐 꺼진다. 정확히는 마우스를 움직이지 않으면 하이라이트가 돌아오지 않는다.

```jsx
export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedLetter, setHighlightedLetter] = useState(null);

  function handleHover(letter) {
    setHighlightedLetter(letter);
  }

  function handleStar(starred) {
    setLetters(letters.map(letter => {
      if (letter.id === starred.id) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter === highlightedLetter
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

따악 보니 첫눈에 문제의 원인이 무엇인지는 일단 알겠다. `highlightedLetter`가 `letter`와 중복된 state이기 때문인 것 같다. 즉 객체 전체를 state로 사용하면 안되고 선택된 letter의  id만 state로 사용해야 하는 것 같다. 그런데 그게 왜 이런 문제를 일으키는거지..? 이유를 알기 위해 다시 위로 올라가서 본문을 읽었다.

그러니까 "star" 혹은 "unstar"  버튼을 누르면 `letters` state가 업데이트 된다. 하지만 `highlightedLetter`는 아직 업데이트 전에 머물러 있고, 비로소 사용자가 마우스를 움직여야 `setHilightedLetter`가 호출되어 업데이트 되는 동기화가 이루어지기 떄문에.
