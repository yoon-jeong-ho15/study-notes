---
date: 2025-08-19
tags:
title: AST
---
참고 : 
[위키피디아](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%EA%B5%AC%EB%AC%B8_%ED%8A%B8%EB%A6%AC)
[https://yceffort.kr/2021/05/ast-for-javascript](https://yceffort.kr/2021/05/ast-for-javascript)

Abstract Syntax Tree, 한국어로 '*추상 구문 트리*'라고 한다.
프로그램 코드를 트리구조로 만들어 구조화한 결과물이다.

```js
const x = 10;

//위의 변수 선언 & 초기화문은 AST로 다음과 같이 변한다.

{
	"type": "VariableDeclaration", // "이건 변수 선언문이야"
	"kind": "const",              // "const 종류의 변수야"
	"declarations": [             // "선언된 변수 목록은 다음과 같아"
		{
		"id": { "name": "x" },     // "변수 이름은 'x'이고"
		"init": { "value": 10 }    // "초기값은 10이야"
		}
	]
}
```

```js
function square(n) {
  return n * n
}

//위의 함수는 다음과 같이 변환된다.
//babel 컴파일러

{
  "type": "Program",
  "start": 0,
  "end": 36,
  "loc": {
    "start": {
      "line": 1,
      "column": 0
    },
    "end": {
      "line": 3,
      "column": 1
    }
  },
  "range": [0, 36],
  "errors": [],
  "comments": [],
  "sourceType": "module",
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 36,
      "loc": {
        "start": {
          "line": 1,
          "column": 0
        },
        "end": {
          "line": 3,
          "column": 1
        }
      },
      "range": [0, 36],
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 15,
        "loc": {
          "start": {
            "line": 1,
            "column": 9
          },
          "end": {
            "line": 1,
            "column": 15
          },
          "identifierName": "square"
        },
        "range": [9, 15],
        "name": "square",
        "_babelType": "Identifier"
      },
      "generator": false,
      "async": false,
      "expression": false,
      "params": [
        {
          "type": "Identifier",
          "start": 16,
          "end": 17,
          "loc": {
            "start": {
              "line": 1,
              "column": 16
            },
            "end": {
              "line": 1,
              "column": 17
            },
            "identifierName": "n"
          },
          "range": [16, 17],
          "name": "n",
          "_babelType": "Identifier"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "start": 18,
        "end": 36,
        "loc": {
          "start": {
            "line": 1,
            "column": 18
          },
          "end": {
            "line": 3,
            "column": 1
          }
        },
        "range": [18, 36],
        "body": [
          {
            "type": "ReturnStatement",
            "start": 22,
            "end": 34,
            "loc": {
              "start": {
                "line": 2,
                "column": 2
              },
              "end": {
                "line": 2,
                "column": 14
              }
            },
            "range": [22, 34],
            "argument": {
              "type": "BinaryExpression",
              "start": 29,
              "end": 34,
              "loc": {
                "start": {
                  "line": 2,
                  "column": 9
                },
                "end": {
                  "line": 2,
                  "column": 14
                }
              },
              "range": [29, 34],
              "left": {
                "type": "Identifier",
                "start": 29,
                "end": 30,
                "loc": {
                  "start": {
                    "line": 2,
                    "column": 9
                  },
                  "end": {
                    "line": 2,
                    "column": 10
                  },
                  "identifierName": "n"
                },
                "range": [29, 30],
                "name": "n",
                "_babelType": "Identifier"
              },
              "operator": "*",
              "right": {
                "type": "Identifier",
                "start": 33,
                "end": 34,
                "loc": {
                  "start": {
                    "line": 2,
                    "column": 13
                  },
                  "end": {
                    "line": 2,
                    "column": 14
                  },
                  "identifierName": "n"
                },
                "range": [33, 34],
                "name": "n",
                "_babelType": "Identifier"
              },
              "_babelType": "BinaryExpression"
            },
            "_babelType": "ReturnStatement"
          }
        ],
        "_babelType": "BlockStatement"
      },
      "_babelType": "FunctionDeclaration"
    }
  ]
}
```
## 생성 과정
1. *토큰화 Tokenization*
	- 프로그래밍 언어의 문법 규칙에 따라 코드를 토큰 단위로 분해
	- 예: `const x = 10;`
	    - `const`: 변수 선언 키워드
	    - `x`: 변수명
	    - `=`: 할당 연산자
		- `10`: 리터럴 값
2. *파싱 Parsing*
	- 토큰들을 문법 규칙에 따라 트리 구조로 조합
	- 각 노드는 코드의 의미적 요소를 나타냄

코드를 이렇게 쪼개서 트리 구조로 만들어야 컴퓨터가 코드를 이해할 수 있다고 한다.
그래서 *IDE*를 사용할 때 IDE가 미리 문법 오류를 알아차릴 수 있게 되는것.
그리고 궁극적으로 *컴파일러*가 고급어 코드를 기계어로 번역하고 프로그램이 컴퓨터에서 실행될 수 있게 된다.
# unifiedjs 에서
unified 라이브러리는 HTML과 Markdown 코드를 AST로 변환하고, AST를 다시 HTML이나 Markdown 코드로 변환하는 과정에서 다양한 처리를 수행할 수 있는 500개가 넘는 라이브러리들의 생태계다.
remark는 md파일을 AST로 다룰 수 있게 하고, rehype는 html파일을 AST로 다룰 수 있게 한다.
둘 사이에 있는 공통된 AST에 대한 규격의 통일(호환)은 모두 unified라는 전체적인 과정안에서 이루어지면서 가능하게 된다.

```ts
const processedContent = await remark()
	.use(breaks)
	.use(remarkMath)
	.use(remarkRehype)
	.use(rehypeKatex)
	.use(rehypeStringify)
	.process(matterResult.content);

const contentHTML = processedContent.toString();
```
이렇게 `use()`를 통해 각 라이브러리들의 기능을 하나씩 추가해 간편하게 사용할 수 있게 된다.
