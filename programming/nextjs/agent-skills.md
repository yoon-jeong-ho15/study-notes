---
title: vercel-labs/agent-skills
date: 2026-01-15
tags:
  - nextjs
  - llm
link:
  - https://github.com/vercel-labs/agent-skills
---
# 개요

vercel에서 클로드 코드(및 다른 코딩 보조 llm)를 위한 skill 문서를 만들어 공개를 했다.

주 내용들로는 **리액트(+ nextjs) 규칙**과 **디자인 규칙**들이 있다.

- `/skills/react-best-practices`
- `/skills/web-design-guidelines`

특정 프로젝트와 글로벌 모두에서 사용 가능하다. 요즘은 안티그래비티+제미나이3을 사용중인데, 꼭 클로드가 아니여도 사용할 수 있다고 하니 궁금해서 받아봤다.
# 규칙 살펴보기
## react-best-practices

`/skills/react-best-practices/rules` 에 보면 리액트 코드 퀄리티를 위한 규칙들이 매우 많이 있다.

크게 주제별로 분류되어 있는데 중요도 순서대로 나열하면 이렇다 :
1. Eliminationg Waterfalls (CRITICAL)
2. Bunde Size Optimization (CRITICAL)
3. Server-Side Performance (HIGH)
4. Client-Side Data Fetching 
5. Re-render Optimization
6. Rendering Performance
7. JavaScript Performance (LOW-MEDIUM)
8. Advanced Patterns (LOW)

### 1. Eliminating Waterfalls

> [!faq] Waterfall 이란? 
> 
> [출처](https://nextjs.org/learn/dashboard-app/fetching-data#what-are-request-waterfalls)