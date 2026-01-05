---
date: 2025-04-02
order: 1
title: 일반 귀납법
---
증명하는 방법으로는 크게 두 가지가 있다.
Indirect proof : 말 그대로 직접적으로 증명하지 않는 증명법. - 반대 명제가 거짓임을 밝혀 이 명제가 참임을 증명하는 귀류법(proof by contradiction)이 대표적인 예시.
Direct proof : 말 그대로 직접적으로 명제가 참임을 증명하는 방법. - 귀납법(induction)이 대표적인 예시.

귀납법은 이산 수학 분야와 컴퓨터 과학에서 매우 중요한 역할을 한다고 한다.
사실 이산 수학에서 *이산 discrete*이라는 특징을 규정하는데 사용된다고 한다.

# 일반 귀납법 Ordinary Induction

P라는 명제를 증명하고자 한다면

1. P(0) 0의 경우에 P가 참이고
2. P(n) 이 참임이 P(n+1)가 참임을 함의한다면
3. P(m)도 마찬가지로 참이다.
   위와 같은 논리 형식에 맞춰서 증명을 하면 된다.
   보기에는 참 간단한데 왜 이렇게 어려운지 모르겠다.

## 예시

수업에서 예시로 제시된 문제는 다음과 같다.

### 1

P = For all $n \in \mathbb{N}$: $1+2+3+\cdots+n=\frac{n(n+1)}{2}$
그럼 위의 P를 증명하기 위해 다음과 같은 두 명제를 증명해야 한다.

1. P(0) 증명 base case
2. P(n)가 P(n+1)를 함의함을 증명
   다음과 같다.

#### 먼저, base case 증명

$\frac{0(0+1)}{2} = 0$

#### 다음, 함의관계 증명

P(n) implies P(n+1)
중요한 부분은 **왼쪽**을 가정하고 **오른쪽**을 증명하는 점이다.
$1+2+3\cdot+n=\frac{n(n+1)}{2}$ 을 가정
$1+2+3\cdot+n+(n+1)=\frac{(n+1)((n+1)+1)}{2}$ 을 증명

P(n)의 양 변은 같기 때문에 (가정되었기 때문에)
양 변에 (n+1)을 더해줘도 양 변은 같다.
$1+2+3\cdot+n+(n+1)=\frac{n(n+1)}{2}+(n+1)$
오른쪽 변은 다음과 같이 변할 수 있다.
$\frac{n(n+1)}{2}+(n+1)=\frac{n(n+1)}{2}+\frac{2n(n+1)}{2}=\frac{n^2+3n+2}{2}=\frac{(n+1)(n+2)}{2}$
마지막으로 (n+2)는 (n+1)+1 이므로 $\frac{(n+1)((n+1)+1)}{2}$가 나온다.

### 2. L트로미노 타일링 문제

$2^n*2^n$개로 이루어진 타일 위에 'ㄴ'자 모양으로 생긴 3칸 넓이의 타일을 배치하려고 할 때,
한 칸은 Bill 이라는 사람의 동상을 세울 수 있다(= 한 칸의 자리는 비어있다).

#### base case

P(0) 는 $2^0*2^0=1$ 이라서 한 칸의 자리가 비어있으니 참

#### inductive step

P(n) -> P(n+1)
$2^{n+1}*2^{n+1}=2^{2n+2}=2^{2n}*2^2=4(2^n*2^n)$
n+1은 n일때의 타일 전체가 4개 있는것과 같다.
그래서 $2^n*2^n$의 경우가 참이라면 나머지 세 개의 $2^n*2^n$에서도 참이고
나머지 세 개에서도 한 자리씩 비게 되고 거기에 마지막 타일을 넣으면 전체$2^{n+1}*2^{n+1}$ 개의 타일에서도 딱 한 칸을 제외하고 모두 타일로 덮을 수 있다.

---

[https://ocw.mit.edu/courses/6-042j-mathematics-for-computer-science-fall-2010/resources/lecture-3-strong-induction/](https://ocw.mit.edu/courses/6-042j-mathematics-for-computer-science-fall-2010/resources/lecture-3-strong-induction/)

> Now, good proofs are very much like good code.
> In fact, one of the reasons we care so much about teaching you how to write a good proof in computer science is so that later on,
> you'll be able to prove that your programs are doing what you expect-- what they're supposed to do.

증명이 프로그래밍에 매우 중요하다고.