---
title: Next.js 튜토리얼
date: 2025-09-23
---
[https://nextjs.org/learn/dashboard-app](https://nextjs.org/learn/dashboard-app)
공식 튜토리얼을 한번 읽어보기로 한다. 
# 1. Getting Started

`/app` : 페이지, 컴포넌트, 로직 등 애플리케이션에 필요한 모든것이 들어있는 경로
`/app/lib` : 재사용할 함수들이 들어있는 경로. (데이터 패칭 등)
`/app/ui` : UI 컴포넌트들
`/public` : 정적 파일들

# 2. CSS Styling

Next.js는 테일윈드 사용을 권장한다.

*CSS모듈*이라는 것도 있는데,
```css
/*home.module.css*/
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```
이렇게 작성되어있는 일반 css파일을
```tsx
import styles from "@/app/ui/home.module.css";

export default function Page(){
	return(
		<main className="flex min-h-screen flex-col p-6">
	      <div className={styles.shape}>
	        <AcmeLogo />
	      </div>
		...      
	)
}
```
이렇게 불러와서 사용할 수 있다.

퀴즈 : CSS모듈의 장점은? 
정답 : Provide a way to make CSS classes *locally scoped* to components by default, reducing the risk of *styling conflicts*.
CSS모듈은 각 컴포넌트마다 고유한 클래스 네임을 생성해 스타일링 충돌을 방지할 수 있다고 한다.


*clsx* 라이브러리도 있다.[깃헙](https://github.com/lukeed/clsx)
이 라이브러리는 조건에 따라 스타일을 주고싶을때(즉 클래스네임을 주고싶을때) 사용한다.

# 3. Optimizing Fonts and Images

# 4. Creating Layouts and Pages
# 5. Navigating Between Pages

Next.js는 기존의 `<a>` 태그 대신 `<Link>`컴포넌트를 사용하기를 권장한다.

`<Link>`컴포넌트는 [*클라이언트 사이드 네비게이션*](https://nextjs.org/docs/app/getting-started/linking-and-navigating#how-routing-and-navigation-works)을 가능하게 해주는데, 이것의 가장 큰 장점은 페이지 이동시마다 **전체 페이지를 로드하지 않는다**는 것이다. 그래서 상태가 지워지지지도, 스크롤 위치가 초기화 되지도 않는다.

## Automatic Code-splitting and prefetching

기존의 리액트 *SPA*는 **처음에 접속**할때 브라우저가 웹앱의 **모든 코드**들을 불러오는 반면, Next.js에서는 자동으로 루트별로 코드들을 분리해(code-splitting) 필요할 때 마다 코드들을 불러온다(prefetching).

이걸 가능하게 하는게 `<Link>` 컴포넌트인데, 브라우저의 뷰포트에 링크 컴포넌트가 나타나면 Next.js는 자동으로 백그라운드에서 해당 링크가 연결하는 페이지의 코드들을 prefetch한다. 그래서 사용자가 링크를 클릭하는 순간에 해당 페이지는 이미 코드를 불러온 상태이니 신속한 페이지 이동을 경험할 수 있다.

정리하자면,

전통적인 서버 렌더링
- 장점 : SEO
- 단점 : 상태 유지 불가(전체 페이지 로드)
위를 극복하기 위한 SPA
- 장점 : 상태 유지, 빠른 페이지 이동
- 단점 : 초기 로딩, SEO불리

이 둘의 장점만을 모은 방법이다.
- prefetching : 빠른 페이지 이동
- code-splitting : 빠른 초기 로딩 
	- Next.js의 고유 기능은 아니고 리액트와 번들러(Webpack, Turbopack)의 기능이라고.
- 서버 렌더링 : SEO에 유리
- 클라이언트 사이드에서의 이동 : 상태 유지
- 
> 스트리밍 streaming 이라는 기능도 Next.js의 네비게이션 경험 향상에 기여를 하는 중요한 기능이지만 이것에 대해서는 나중에 데이터 패칭 부문에서  다룬다.

# 6. Setting Up Your Database

# 7. Fetching Data

데이터 패칭, 즉 데이터베이스에서 데이터를 조회할 때에는 다음과 같은 고려 사항들이 있다.

1. API 레이어 (route handler) [링크](https://nextjs.org/docs/app/api-reference/file-conventions/route)
2. Database 쿼리
3. 서버 컴포넌트
4. SQL

## API 레이어

클라이언트 측에서 바로 DB에 접근해 데이터를 조회하는건 보안상 문제가 있을 수 있다.
- DB 연결 정보 (api key, 비밀번호 등) 노출
- SQL injection 공격에 취약

그래서 전통적으로 중간에 백엔드 서버가 클아이언트의 요청을 검증하고 요청한 데이터를 보내준다.
Next.js는 풀스텍 프레임워크다. **Node.js 런타임**에서 실행되는 서버에서 이 요청을 처리한다.
`fetch("/dir1/dir2",{...})`라고 클라이언트에서 요청한다면 `app/dir1/dir2/route.ts`파일을 작성해둔다.
spring boot에서 **컨트롤러**라고 불렀던 이 모듈을 Next.js에서는 **route handler**라고 부른다.

> spring boot는 MVC 패턴을 따라 애플리케이션을 설계하도록 되어있지만, Next.js는 반드시 그럴 필요는 없다.













