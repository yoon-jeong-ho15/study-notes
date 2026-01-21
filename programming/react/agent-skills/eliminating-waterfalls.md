---
title: Eliminating Waterfalls
date: 2026-01-15
order: 1
---
# 1. Eliminating Waterfalls

중요도 : CRITICAL (워터폴 현상은 가장 큰 성능 저하 요인이다.)

> [!faq] Waterfall 이란? 
> 이전 요청이 완료되어야 다음 요청이 진행되는 것을 말한다. 즉 여러 네트워크 요청이 있을때 순차적으로 하나씩 진행되는 현상을 의미하는데, `await`을 사용하다보면 발생할 수 있다고 한다.
> 물론 의도적으로 워터폴이 일어나게 설계해야할 수도 있지만, 그렇지 않은 경우가 더 많다.
> [출처](https://nextjs.org/learn/dashboard-app/fetching-data#what-are-request-waterfalls)

## 1. Defer Await Until Needed

중요도 : HIGH

가장 간단하지만 효과 등급은 HIGH 이다. 
스킵되는 경우가 잦은 설계, 미뤄진 작업이 무거운 경우에 특히 효과적이다.
### Incorrect

```tsx
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // Returns immediately but still waited for userData
    return { skipped: true }
  }
  
  // Only this branch uses userData
  return processUserData(userData)
}
```

### Correct

```tsx
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // Returns immediately without waiting
    return { skipped: true }
  }
  
  // Fetch only when needed
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

## 2. Dependency-Based Parallelization

**중요도 : CRITICAL** (2-10배 속도 개선)

*Partial Dependencies 부분적 의존성*이란 여러 비동기 작업이 있을 때, 어떤 작업은 다른 작업의 결과가 필요하지만, 어떤 작업은 필요하지 않은 경우를 말한다. 

- `fetchUser()` : 1초 소요
- `fetchConfig()` : 10초 소요
- `fetchProfile(user.id)` : 10초 소요

이 세 작업이 있을때 `fetchProfile(user.id)`는 `fetchUser()`의 결과를 필요로 한다. 즉 이 경우에는 워터폴이 필요한 상황.
### Incorrect

그래서 위의 두 작업은 `Promise.all()`로 한번에 가져오고서 그 후에 아래의 작업을 실행하도록 코드를 작성하는게 흔한 패턴이다. 

`profile`이 필요로하는 `user`를 가져오는데 1초가 걸리는데 쓸데없이 `config` 까지 기다리면서 10초나 기다리는 상황. 

```tsx
const [user, config] = await Promise.all([
  fetchUser(),  // 1s
  fetchConfig() // 10s
]) // 10s
const profile = await fetchProfile(user.id) // 10s
// total : 20s
```

`user`만 먼저 가져오는 방법으로 해결할 수 있다.

```tsx
const user = await fetchUser(); // 1s
const [profile, config] = await Promise.all([
  fetchProfile(user.id),  // 10s
  fetchConfig() // 10s
]) // 10s

// total : 11s
```

그런데 문제는 네트워크 연결 상태 변동과 같은 어떤 이유들로 인해 위 작업들의 소요 시간이 바뀐다면? 아래의 코드가 위의 코드보다 오히려 시간이 더 걸릴수도 있게 된다.

따라서 모든 상황에서 항상 최소한의 시간만 사용하고 싶다면 다음과 같이 **직접 의존 관계를 포함해 작성**해야 한다.

```tsx
const [[user, profile], config] = await Promise.all([
  fetchUser().then(user=> fetchProfile(user.id).then(profile => [user, profile])),
  fetchConfig()
])
```

뭐가 문제인가 싶지만, **(역시나 다른 모든 문제들처럼)** 프로젝트의 규모가 거대해지면 이렇게 작성하는건 힘들다고 한다. [링크](https://github.com/shuding/better-all/discussions/3)

이렇게 위와 비교해 조금만 복잡해져도 가독성이 크게 떨어진다. 

```tsx
const promiseA = getA()
const promiseB = getB()
const promiseC = getC()
const promiseD = Promise.all([promiseA, promiseC]).then(([a, c]) => getD(a, c))
const promiseE = Promise.all([promiseB, promiseC]).then(([b, c]) => getE(b, c))
const [a, b, c, d, e] = await Promise.all([promiseA, promiseB, promiseC, promiseD, promiseE])
```

### Correct

`better-all` 라이브러리를 사용하면 간단히 해결된다.

```tsx
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})
```

```tsx
const { a, b, c, d, e } = await all({
  async a() { return getA() },
  async b() { return getB() },
  async c() { return getC() },
  async d() { return getD(await this.$.a, await this.$.c) },
  async e() { return getE(await this.$.b, await this.$.c) }
})
```


[better-all 라이브러리 링크](https://github.com/shuding/better-all)

## 3. Prevent Waterfall Chains in API Routes

**중요도 : CRITICAL** (2-10배 속도 개선)

api 루트나 서버액션을 사용할 때, `await` 없이 `Promise`를 사용한다. 물론 복잡한 경우에는 위에서 제시한 `better-all` 라이브러리 사용하는게 더 적합한 해결법.
### Incorrect

**워터폴의 전형**적인 예시. `fetchConfig()`는 자신과 아무 상관 없는 `auth()`을 가디라기고 `fetchData()`도 자신과 상관 없는 `fetchConfig()`를 기다린다.

```tsx
export async function GET(request: Request) {
  const session = await auth() // 1s
  const config = await fetchConfig() // 10s
  const data = await fetchData(session.user.id) // 10s
  return Response.json({ data, config })
  
  // total 21s
}
```

### Correct


```tsx
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise // 1s
  const [config, data] = await Promise.all([
    configPromise, // 10s
    fetchData(session.user.id) // 10s
  ])
  return Response.json({ data, config })
}
// total 11s
```

## 4. Promise.all() for Independent Operations

**중요도 : CRITICAL** (2-10배 속도 개선)

`async` 작업들 끼리 의존하고 있지 않을때(2번과 같은 경우가 아닐 때)는 `Promise.all()`을 사용한다.

```tsx
// Incorrect
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()

// Correct
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

## 5. Strategic Suspense Boundaries

중요도 : HIGH

`Suspense`랩퍼 컴포넌트를 사용한다. 이러면 데이터를 기다리는 동안은 미리 정의해둔 `Skeleton` 이 보이게 되고 더 빠른 초기 페인팅을 통해 훨씬 나은 사용자 경험을 제공한다.

### Incorrect

```tsx
async function Page() {
  const data = await fetchData() // Blocks entire page
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

### Correct

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // Only blocks this component
  return <div>{data.content}</div>
}
```

하지만 무작정 사용하면 안되고 다음과 같은 경우에는 `Suspense` 사용을 하면 안된다.
- 레이아웃을 결정할 때 꼭 필요한 데이터인 경우.
- SEO에 중요한 데이터인 경우.
- 데이터 크기가 작아 굳이 비용을 소모해가며 스켈레톤을 보여줄 필요가 없는 경우.
- 순간 튀는 레이아웃 변동 layout shift, content jumping을 방지해야 하는 경우 : **Trade-off** -  빠른 페이지 로딩을 챙기지만 potential layout shift를 감수할것인가 아니면 느리더라도 안정적인 Ui를 만들것인가 

> [!faq] 결합도 문제
> 위와 같은 설계대로면 `DataDisplay` 컴포넌트의 테스트 복잡도가 올라가지 않을까? 


