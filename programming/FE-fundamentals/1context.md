---
title: "가독성 1 : 맥락 줄이기"
date: 2025-12-03
tags:
order: 1
---
# 맥락 줄이기

## 1. 같이 실행되지 않는 코드 분리하기

### 문제 코드

아래의 예시에서 **같이 실행되는 코드**는 무엇이고 **같이 실행되지 않는 코드**는 무엇일까?

```tsx
function SubmitButton() {
  const isViewer = useRole() === "viewer";

  useEffect(() => {
    if (isViewer) {
      return;
    }
    showButtonAnimation();
  }, [isViewer]);

  return isViewer ? (
    <TextButton disabled>Submit</TextButton>
  ) : (
    <Button type="submit">Submit</Button>
  );
}
```

위의 `SubmitButton` 컴포넌트는 `isViewer` 에 따라 **다른 행동**을 하는 컴포넌트이다.
- 참 : `TextButton` 렌더링.
- 거짓 : `showButtonAnimation` 호출, `Button` 을 렌더링.

즉 '참 or 거짓' 에 따라 **어떤 코드는 전혀 실행되지 않는다**는 얘기다. 

### 개선 코드

```Tsx
function SubmitButton() {
  const isViewer = useRole() === "viewer";

  return isViewer ? <ViewerSubmitButton /> : <AdminSubmitButton />;
}

function ViewerSubmitButton() {
  return <TextButton disabled>Submit</TextButton>;
}

function AdminSubmitButton() {
  useEffect(() => {
    showButtonAnimation();
  }, []);

  return <Button type="submit">Submit</Button>;
}
```

위 처럼 `isViewer` 가 참일때만 실행되는 코드와 거짓일때만 실행되는 코드를 분리하여 **별도의 컴포넌트**로 만들면 훨씬 가독성이 개선된다.

## 2. 구현 상세 추상화하기

### 문제 코드

아래의 예시는 한 컴포넌트 안에 상세 로직과 UI 렌더링 책임이 함께 있는 경우다.

```Tsx
function FriendInvitation() {
  const { data } = useQuery(/* 생략.. */);

  // 이외 이 컴포넌트에 필요한 상태 관리, 이벤트 핸들러 및 비동기 작업 로직...

  const handleClick = async () => {
    const canInvite = await overlay.openAsync(({ isOpen, close }) => (
      <ConfirmDialog
        title={`${data.name}님에게 공유해요`}
        cancelButton={
          <ConfirmDialog.CancelButton onClick={() => close(false)}>
            닫기
          </ConfirmDialog.CancelButton>
        }
        confirmButton={
          <ConfirmDialog.ConfirmButton onClick={() => close(true)}>
            확인
          </ConfirmDialog.ConfirmButton>
        }
        /* 중략 */
      />
    ));

    if (canInvite) {
      await sendPush();
    }
  };

  // 이외 이 컴포넌트에 필요한 상태 관리, 이벤트 핸들러 및 비동기 작업 로직...

  return (
    <>
      <Button onClick={handleClick}>초대하기</Button>
      {/* UI를 위한 JSX 마크업... */}
    </>
  );
}
```

이 컴포넌트는 지금 여러 *맥락*을 가지고 있다.

### 개선 코드 

```Tsx
export function FriendInvitation() {
  const { data } = useQuery(/* 생략.. */);

  // 이외 이 컴포넌트에 필요한 상태 관리, 이벤트 핸들러 및 비동기 작업 로직...

  return (
    <>
      <InviteButton name={data.name} />
      {/* UI를 위한 JSX 마크업 */}
    </>
  );
}

function InviteButton({ name }) {
  return (
    <Button
      onClick={async () => {
        const canInvite = await overlay.openAsync(({ isOpen, close }) => (
          <ConfirmDialog
            title={`${name}님에게 공유해요`}
            cancelButton={
              <ConfirmDialog.CancelButton onClick={() => close(false)}>
                닫기
              </ConfirmDialog.CancelButton>
            }
            confirmButton={
              <ConfirmDialog.ConfirmButton onClick={() => close(true)}>
                확인
              </ConfirmDialog.ConfirmButton>
            }
            /* 중략 */
          />
        ));

        if (canInvite) {
          await sendPush();
        }
      }}
    >
      초대하기
    </Button>
  );
}
```



## 로직 종류에 따라 합쳐진 함수 쪼개기

## 2. 이름 붙이기
### 복잡한 조건에 이름 붙이기
### 매직 넘버에 이름 붙이기
## 3. 위에서 아래로 읽히게 하기
### 시점 이동 줄이기
### 삼항 연산자 단순하게 하기

