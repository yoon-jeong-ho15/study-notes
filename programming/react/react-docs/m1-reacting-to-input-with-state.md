---
date: 2026-01-30
title: Reacting to Input with State
link:
  - https://react.dev/learn/reacting-to-input-with-state
---
### You will learn

- 선언형 프로그래밍 방식과 명령형 방식의 차이
- 컴포넌트가 가져야 할 시각적 state들을 나열하는 법
- 한 시각적 state에서 다른 시각적 state로 넘어가는 방법

## 선언형 UI 

UI를 생각할 때 우리들은 보통 사용자들의 행동에 맞춰 변화하는것이라 생각한다. 가령 사용자가 폼을 제출하는 경우를 생각해보자 :
- 사용자가 입력창에 글자를 적어야 전송 버튼이 활성화 된다.
- 전송 버튼을 누르면 로딩을 알리는 스피너와 함께 잠시 비활성화 된다.
- 요청이 완료되면 폼이 사라지고, 제출 완료를 알리는 안내문구가 나타난다.
- 요청 처리에 오류가 발생하면, 에러 알림이 나타난다.

리액트 이전의 명령형 프로그래밍 체계에서, 위의 사항들은 모두 개발자에 의해 구현되어야 할 사항이다. 즉 자바스크립트로 일일이 코드를 작성했어야 했다는 말이다. 명령형 프로그래밍은 마치 택시에 타서 택시 기사에게 목적지를 말해주는 대신 그때마다 우회전인지 좌회전인지 직진해야하는지를 말해주는것과 같다. 

위의 폼 제출 상황을 명령형 코드로 작성하면 아래와 같다. 

```js
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

간단한 폼 제출 UI만 해도 코드가 이렇게 길어지는데 프로그램의 규모가 거대해지면 그 코드의 복잡도는 얼마만할까?

리액트는 이 문제를 해결해준다. 선언형 UI는 각 UI가 어때야 하는지를 정하면 리액트가 각 화면을 어떻게 구현할것인지 스스로 해결한다. 택시에 타서 목적지만 말하면 되는것처럼.

## 선언형 방식으로 UI 작성하기

1. Identify : 컴포넌트의 각 시각적 상태를 결정한다.
2. Determine : 상태의 변화를 일으킬 것이 무언인지 결정한다.
3. Represent : `useState`로 메모리의 상태를 표현한다.
4. Remove : 필요 없는 상태 변수를 제거한다.
5. Connect : 상태를 변경하는 이벤트 핸들러를 연결한다.

### 1.  Identify : 컴포넌트의 각 시각적 상태를 결정한다.

UI 컴포넌트가 어떤 모습을 가질것인지 먼저 생각해야 한다. 가령 폼 제출에 있어서는 다음의 상태들이 있을것이다 : 
- empty - 제출 버튼 비활성화
- typing - 제출 버튼 활성화
- submitting - 입력과 제출 버튼 비활성화
- success - 전체 폼이 사라지고 안내 문구 출력
- error - typing상태와 같지만 에러 문구도 함께 출력

### 2. Determine : 상태의 변화를 일으킬 것이 무언인지 결정한다.

state를 변화시키는 트리거는 두 가지가 있다 :
- 사용자 입력 - 마우스 클릭, 키보드 입력, 링크 이동 등
- 컴퓨터 입력 - 네트워크 응답 도착, 타임아웃, 이미지 로딩 등

이제 각 입력들이 어떻게 상태를 변화하는지 결정한다.
- 입력창이 비어있다면 empty 상태, 비어있지 않다면 typing 상태일 것이다.
- 사용자가 제출 버튼을 누르면 submitting 상태가 될 것이다.
- 요청이 성공적으로 처리되었다는 네트워크 응답이 도착하면 success상태로, 반대로 오류가 발생했다면 error 상태가 될 것이다.

### 3. Represent : `useState`로 메모리의 상태를 표현한다.

다음으로 각 컴포넌트의 시각적 state를 `useState`로 표현해야 한다. 

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);

const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```
### 4. Remove : 필요 없는 상태 변수를 제거한다.

위에 작성한 state들을 검토해보자. 다음의 질문들을 통해서 :

- **state가 역설을 일으키는가?** - 가령 `isTyping`과 `isSubmitting`이 동시에 `true`일 수는 없다. **이러한 역설은 보통 state가 충분히 제한되지 않았음을 의미한다.** 이 두 state는 네 가지 조합을 가지지만(참-참, 참-거짓, 거짓-참, 거짓-거짓) 유효한 것은 세 개 뿐이다(둘 다 참인 경우를 제외해서). 이런 경우 이 state들을 합쳐서 세 값(`isTyping`, `isSubmitting`, `success`)을 가지는 하나의 state로 만들 수 있다. 
- **같은 정보가 이미 다른 state에 있는가?** - 위와 비슷한 경우로, `isEmpty`와 `isTyping`은 동시에 참일 수 없다. 이런 경우에 **두 state를 별도로 분리하면 버그**가 발생할 수 있다. 이 경우에 `isEmpty`는 `answer.length===0`과 같기 때문에 `isEmpty` state를 제거할 수 있다.
- **다른 state를 바꿔서 똑같은 정보를 얻을 수 있는가?** - `isError` 라는 state는 `error !=== null`과 같기 때문에 `isError` 대신 `error` 를 사용한다. 

이 리팩토링의 결과로 state는 이렇게 정리될 수 있다.

```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
```

요약 :
- `isEmpty`삭제.
- `isTyping`,`isSubmitting`,`success`를 하나의 상태로 합쳤다. 
- `isError` 삭제.
### 5. Connect : 상태를 변경하는 이벤트 핸들러를 연결한다.

이제 state들을 정리했으니 이 state들을 설정하는 이벤트 핸들러를 연결해야 한다.

```jsx
export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      //...
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}

```

이 코드만 보면 사실 위의 명령형 코드보다 코드량은 많지만, 훨씬 가독성이 좋고 그러기 때문에 덜 연약(허술)하다. 그리고 코드의 추가와 변경도 용이해진다.

## 예시 문제

### 1. CSS 클래스 토글

```jsx
export default function Picture() {
  return (
    <div className="background background--active">
      <img
        className="picture" // picture--active 
        alt="Rainbow houses in Kampung Pelangi, Indonesia"
        src="https://i.imgur.com/5qwVYb1.jpeg"
      />
    </div>
  );
}
```

이미지를 누르면 배경이 사라지고(`backgroud--active`클래스를 제거) 이미지에 `picture--active`라는 클래스를 추가하게 만들고, 반대로 다시 배경을 누르면 `background--active` 클래스를 추가하고 `picture--active` 클래슬르 제거하도록 수정해야한다.

```jsx
export default function Picture() {
  const [element,setElement] = useState("");
  
  return (
    <div className={`background ${element==="background" ? "background--active" : ""}`}
      onClick={()=>setElement("background")}>
      <img
        className={`picture ${element==="image" ? "picture--active":""}`}
        alt="Rainbow houses in Kampung Pelangi, Indonesia"
        src="https://i.imgur.com/5qwVYb1.jpeg"
        onClick={(e)=>{
          e.stopPropagation();
          setElement("image");}}
      />
    </div>
  );
}
```

답안과 다르지만 나는 이렇게 구현했다.

### 3. 명령형 코드 리팩토링

재밌게 리액트가 아닌 일반 자바스크립트 코드를 수정하는 문제. 내부적으로 리액트가 어떻게 작동하는지에 대한 힌트를 준다.

```jsx
let firstName = 'Jane';
let lastName = 'Jacobs';
let isEditing = false;

function handleFormSubmit(e) {
  e.preventDefault();
  setIsEditing(!isEditing);
}

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

function setFirstName(value) {
  firstName = value;
  updateDOM();
}

function setLastName(value) {
  lastName = value;
  updateDOM();
}

function setIsEditing(value) {
  isEditing = value;
  updateDOM();
}

function updateDOM() {
  if (isEditing) {
    editButton.textContent = 'Save Profile';
    // TODO: show inputs, hide content
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Edit Profile';
    // TODO: hide inputs, show content
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
  // TODO: update text labels
  firstNameText.textContent = firstName;
  lastNameText.textContent = lastName;
  helloText.textContent = (
    'Hello ' +
    firstName + ' ' +
    lastName + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;

```

`editButton`을 클릭 -> `handleFormSubmit` -> `setIsEditing` -> `<b>`태그 숨기고 `input` 태그 노출 -> `oninput` 핸들러 -> 글자 입력시마다 `updateDOM` 호출 -> `firstNameText.textContent = firstName` 으로 아래에 출력되는 문구 실시간 업데이트

이런 흐름으로 실행되는데 정말 리액트 전에는 이걸 자바스크립트로 하나하나 작성했어야 했구나.. 놀랐다.

