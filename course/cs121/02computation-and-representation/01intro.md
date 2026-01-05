---
title: 표현
date: '2025-07-31'
order: 1
---

# 개요
The main takeaways from this chapter are:

- We can represent all kinds of objects we want to use as inputs and outputs using _binary strings_. For example, we can use the __binary basis__ to represent integers and rational numbers as binary strings (see [Section 2.1.1](https://introtcs.org/public/lec_02_representation.html#naturalnumsec) and [Section 2.2](https://introtcs.org/public/lec_02_representation.html#morerepressec)).
- We can _compose_ the representations of simple objects to represent more complex objects. In this way, we can represent lists of integers or rational numbers, and use that to represent objects such as matrices, images, and graphs. __Prefix-free encoding__ is one way to achieve such a composition (see [Section 2.5.2](https://introtcs.org/public/lec_02_representation.html#prefixfreesec)).
- A _computational task_ specifies a map from an input to an output— a __function__. It is crucially important to distinguish between the “what” and the “how”, or the _specification_ and _implementation_ (see [Section 2.6.1](https://introtcs.org/public/lec_02_representation.html#secimplvsspec)). A __function__ simply defines which output corresponds to which input. It does not specify __how__ to compute the output from the input, and as we’ve seen in the context of multiplication, there can be more than one way to compute the same function.
- While the set of all possible binary strings is infinite, it still cannot represent __everything__. In particular, there is no representation of the _real numbers_ (with absolute accuracy) as binary strings. This result is also known as “Cantor’s Theorem” (see [Section 2.4](https://introtcs.org/public/lec_02_representation.html#cantorsec)) and is typically referred to as the result that the “reals are uncountable.” It is also implied that there are **different levels** of infinity though we will not get into this topic in this book (see [Remark 2.10](https://introtcs.org/public/lec_02_representation.html#generalizepowerset)).

The two “big ideas” we discuss are [Big Idea 1](https://introtcs.org/public/lec_02_representation.html#representtuplesidea) - we can compose representations for simple objects to represent more complex objects and [Big Idea 2](https://introtcs.org/public/lec_02_representation.html#functionprogramidea) - it is crucial to distinguish between _functions_ (“what”) and _programs_ (“how”). The latter will be a theme we will come back to time and again in this book.

이번 장에서는 이 네 가지에 대해서 배운다고 한다.  
  
- '모든 종류'의 자료를 binary strings("01110001110")로 표현이 가능하다는 사실.  
- prefix-free encoding이라는 방법을 통해 복잡한 자료를 포현한다는 사실. (복잡한걸 이진법으로 표현할 수 있는 더 단순한 것들의 묶음으로 표현)  
- 컴퓨터가 하는일은 함수다. 입력 -> 출력을 맵핑하는 일.   
- 이진법의 조합은 무한하지만, 그럼에도 불구하고 세상의 모든것을 표현할 수는 없다는 사실. (칸토어의 이론)


## "표현"
우리가 컴퓨터에 사진, 음악, 영상 등 데이터를 저장할 때 진짜로 저장되는 것은 그것들의 '표현'이다. 우리의 뇌 마저도 감각된 그 자체가 입력되지 않고 그것의 표현을 저장한다.  
컴퓨터는 0과 1로 된 binary strings밖에 저장하지 못하니까 결국 모든걸 0과 1로 번역하고 그걸 저장한다.  
  
그래서 중요한건 "어떻게"의 문제다. representation scheme.  
자연수, 소수, 문자열, 이미지, 영상 등등 모든걸 어떻게 0과1로 만들것인가?