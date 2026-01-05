---
title: 칸토어 정리
date: 2025-07-31
order: 3
---

위의 수업내용 정리를 보면 0과1의 조합은 무한하지만, 그럼에도 불구하고 모든걸 다 0과1로 표현할 수 없다고 했다. (위의 예시에서도 봤듯 12.3을 0과1만으로 표현해내지 못했기 때문에)  
그러면서 교재는 칸토어의 집합론을 소개하는데, 철학수업때 종종 들어봐서 익숙한 이름이었다.
# Countable sets 가산집합
어떤 집합 S가 가산집합이란건 무슨 의미냐면, 여기에 대응하는 onto map C가 있다는 것이다. 어떤 집합 A에서 C라는 함수를 사용하면 S의 모든 요소들이 A의 요소들과 맵핑이 된다면 가산 집합이라는 뜻.  
자연수가 대표정인 가산 집합니다. 0과1을 조합해서 모든 숫자를 나타낼 수 있으니까. 즉 위에서 본 NtS 함수가 있으니까.  
그런데 실수를 이진수로 표현하는 방법에서 본것처럼 어떤 실수는 0과 1만으로 표현할 수 없다. 그래서 실수들의 집합은 불가산 집합. 그리고 불가산 집합은 가산 집합보다 크다.
```python
def diagonalize(f,n):
    """Input: function f mapping integers to sequences of 0 and 1. A number n
       Output: a sequence L of length n that is not in the image of f on {0....n-1}
    """
    return [1-f(k)[k] for k in range(n)]
#%%

# Example:
def f(k): # some way to map integers to lists of length 100
    return [(k * i) % 2 for i in range(100)]
    
D = diagonalize(f,100)
print(D)

#%%
found = False
for k in range(100):
    if D == f(k):
        print("D is in f's image")
        found = True
        break

if not found: print("D is not in f's image")
```

diagonalize()는 0부터 n까지  
f(k)함수를 실행해서 1에서 그것을 뺀 값들을 반환한다.  
f(k)이 호출되면  
0부터 100까지 반복하면서 k와 곱해서 나온 수를 mod 2 한 결과를 반환한다.  
  
f(0)은 k가 0이니 모든 숫자가 다 0.  
f(0)[0]은 0, 1-1=1.  
  
f(1)은 1,2,3,4,5, ..., 100들을 다 %2 하면 1,0,1,0,...0이 된다.  
f(1)[1]은 1, 1-1=0.  
  
f(2)는 2,4,6,8, ...., 200 이고 다 %2 해보면 0,0, ...., 0.  
f(2)[2]는 0, 1-0=1.  
  
그렇게 D에는 [1,0,1, .....] 이 담긴다.  
  
그리고 검증단계 k=0 부터 100까지  
f(0)은 [0,0,0, .... , 0] != D.  
f(1)은 [1,0,1,0,....., 0] != D.  
  
그러니까 f가 100가지 다른 방법으로 0~100의 자연수를 0과1로 표현한거라면  
D는 각f(k)[k]의 자리 숫자의 반대되는 숫자를 가지는, 기존에 없던 새로운 표현이다.
