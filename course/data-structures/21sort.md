---
title: 정렬
date: 2025-09-08
order: 21
tags:
  - 버블정렬
  - 선택정렬
  - 삽입정렬
  - 병합정렬
  - 힙정렬
  - 퀵정렬
---
## 분류
**비교 횟수**에 의해
- 비교에 기반한 정렬 알고리즘에서는 최선의 경우 $O(nlogn)$, 최악의 경우 $O(n^2)$.
교환Swap 횟수에 의해
**메모리 사용**에 의해
- 어떤 알고리즘은 제자리(in-place)에서 수행되므로 데이터를 임시적으로 저장할 보조 공간을 위해 O(1)이나 O(logn) 메모리가 필요하다.
재귀에 의해
안정성에 의해
**적응성Adaptability** 에 의해
- 어떤 알고리즘들은 *선정렬presortedness* 에 의해 복잡도가 바뀐다.

## 대표적인 정렬 알고리즘들
### 버블 정렬 Bubble Sort
```c
void BubbleSort(int arr[], int len) {
  int i, j, tmp;
  for (i = 0; i < len - 1; ++i) {
    for (j = 0; j < len - i - 1; ++j) {
      if (arr[j] > arr[j + 1]) {
        tmp = arr[j];
        arr[j] = arr[j+1];
        arr[j+1] = tmp;
      }
    }
  }
}
```

```c
void BubbleSort(int arr[],int len){
	int pass, i, temp, swappted = 1;
	for(pass=len-1;pass>=0 && swapped;pass--){
		swapped = 0;
		for(i=0;i<pass;i++){
			if(arr[i] > arr[i+1]){
				temp = arr[i];
				arr[i] = arr[i+1];
				arr[i+1] = temp;
				swapped = 1;
			}
		}
	}
}
```

아래의 버블 정렬은 **조기 종료 조건**을 추가해서 약간은 최적화 된 버전.

### 선택 정렬 Selection Sort

```c
void selectionSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
      
        int min_idx = i;
        
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[min_idx]) {
              
                min_idx = j;
            }
        }
        
        int temp = arr[i];
        arr[i] = arr[min_idx];
        arr[min_idx] = temp;
    }
}
```

버블정렬과 달리 *조기 종료 조건*을 추가할 수 없다. 

### 수업 외 내용
수업에서는 다루지 않았지만 궁금해서 생각해 본 것들이다.

#### 버블 정렬과의 비교 : 조기 종료 가능성의 차이

**버블 정렬에서 조기 종료가 가능한 이유**:
버블 정렬은 인접한 원소들끼리 비교하여 자리를 바꾸는 방식으로 진행되는 알고리즘이다. i 순회마다 j 순회를 반복하는 것은 선택 정렬과 동일하지만, 버블 정렬에서는 j 순회 동안 여러 번의 인접한 두 요소 교환이 이루어진다.
**핵심은 j 순회 때 한 번도 교환이 이루어지지 않았다는 것이 이미 해당 배열이 정렬되었음을 의미한다는 점이다.** 이 조건에 해당하는 불린 변수를 선언하여 사용한다면, 반복문의 조건에 추가하는 방식으로 조기 종료를 구현할 수 있다.

**선택 정렬에서 조기 종료가 어려운 이유**:
선택 정렬은 각 순회에서 최솟값을 찾은 후 **단 한 번만 교환**한다. 현재 위치에 올바른 값이 이미 있어도 전체 배열을 스캔해야만 그것이 최솟값임을 확인할 수 있다.

#### 성능 비교

그렇다면 버블 정렬이 선택 정렬보다 무조건 우월한 정렬 알고리즘일까? 그렇지 않다. 버블 정렬에서는 잦은 교환 연산이 이루어지기 때문이다.
**교환 횟수 차이**:
- **선택 정렬**: O(n) - 각 순회마다 최대 1번의 교환
- **버블 정렬**: O(n²) - 인접 요소들의 빈번한 교환
교환 연산은 비교 연산보다 비용이 크므로, 적은 교환 횟수를 가진 선택 정렬이 일반적으로 더 효율적이다.
#### *조기 종료 조건*을 생각해볼 수 있을까?
```js
function selectionSort(arr) {
    for (let i = 0; i < arr.length; i++) {      
        let min_idx = i;
        let check = true;
        
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[min_idx]) {
                min_idx = j;
            }
            if(arr[j]>arr[j+1]){
	            check = false;
            }
        }
        
        if(min_idx === i && check){
	        break;
        }else{
	        let temp = arr[i];
	        arr[i] = arr[min_idx];
	        arr[min_idx] = temp;
        }
    }
}
```

동작 예시
`[5, 3, 1, 7, 9]` 배열 정렬:
1. 첫 번째 순회: 1을 찾아서 맨 앞으로 → `[1, 3, 5, 7, 9]`
2. 두 번째 순회: 3이 이미 최솟값이고, 나머지가 정렬됨을 확인 → **조기 종료**

**한계점 - 추가 비교 연산의 오버헤드**:
- 기존 선택 정렬: 최솟값 찾기 연산만 수행
- 개선된 버전: 최솟값 찾기 + 정렬 상태 확인 연산
- **최악의 경우**: 비교 연산이 약 2배로 증가

**최악의 경우들**:
1. **역순으로 정렬된 배열**: `[9, 7, 5, 3, 1]`
    - 매번 교환이 필요하고, 정렬 상태 확인도 항상 실패
2. **거의 역순인 배열**: `[8, 7, 6, 5, 1, 2, 3, 4]`
    - 중간까지 정렬 확인이 계속 실패
3. **무작위 배열**: 정렬 상태가 후반부에서야 달성되는 경우
#### 결론
선택 정렬의 조기 종료 최적화는 **이미 정렬된 데이터나 부분적으로 정렬된 데이터**에서만 효과적이다. 일반적인 경우에는 추가 비교 연산으로 인한 오버헤드가 발생할 수 있어, 실용성은 제한적.

> 어떤 경우들이 최악의 경우들일까? 

### 삽입 정렬 Insertion Sort

```c
void insertionSort(int arr[], int n) {
    for (int i = 1; i < n; ++i) {
        int key = arr[i];
        int j = i - 1;

        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
    }
}
```
장점 :
- 간단한 구현
- 작은 데이터에 효율적
- 적응적 : 입력 리스트가 이비 정렬되어있다면, 삽입 정렬은 d가 반번의 횟수일 때 O(n+d)의 시간이 걸린다.
- 온라인 : 삽입 정렬은 입력 리스트를 받으면서 정렬할 수 있다.

### 병합 정렬 Merge Sort

Divide-and-Conquere의 대표적인 예시.

```c
#include <stdio.h>
#include <stdlib.h>

// Merges two subarrays of arr[].
// First subarray is arr[l..m]
// Second subarray is arr[m+1..r]
void merge(int arr[], int l, int m, int r){
    
    int i, j, k;
    int n1 = m - l + 1;
    int n2 = r - m;

    // Create temp arrays
    int L[n1], R[n2];

    // Copy data to temp arrays L[] and R[]
    for (i = 0; i < n1; i++)
        L[i] = arr[l + i];
    for (j = 0; j < n2; j++)
        R[j] = arr[m + 1 + j];

    // Merge the temp arrays back into arr[l..r
    i = 0;
    j = 0;
    k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        }
        else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    // Copy the remaining elements of L[],
    // if there are any
    while (i < n1) {
        arr[k] = L[i];
        i++;
        k++;
    }

    // Copy the remaining elements of R[],
    // if there are any
    while (j < n2) {
        arr[k] = R[j];
        j++;
        k++;
    }
}

// l is for left index and r is right index of the
// sub-array of arr to be sorted
void mergeSort(int arr[], int l, int r){
    
    if (l < r) {
        int m = l + (r - l) / 2;

        // Sort first and second halves
        mergeSort(arr, l, m);
        mergeSort(arr, m + 1, r);

        merge(arr, l, m, r);
    }
}

// Driver code
int main(){
    
    int arr[] = {38, 27, 43, 10};
    int arr_size = sizeof(arr) / sizeof(arr[0]);

    mergeSort(arr, 0, arr_size - 1);
    int i;
    for (i = 0; i < arr_size; i++)
        printf("%d ", arr[i]);
    printf("\n");
    
    return 0;
}
```

### 힙 정렬 Heap Sort

```c
// To heapify a subtree rooted with node i
// which is an index in arr[].
void heapify(int arr[], int n, int i) {

    // Initialize largest as root
    int largest = i; 

    // left index = 2*i + 1
    int l = 2 * i + 1; 

    // right index = 2*i + 2
    int r = 2 * i + 2;

    // If left child is larger than root
    if (l < n && arr[l] > arr[largest]) {
        largest = l;
    }

    // If right child is larger than largest so far
    if (r < n && arr[r] > arr[largest]) {
        largest = r;
    }

    // If largest is not root
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;

        // Recursively heapify the affected sub-tree
        heapify(arr, n, largest);
    }
}

// Main function to do heap sort
void heapSort(int arr[], int n) {

    // Build heap (rearrange array)
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }

    // One by one extract an element from heap
    for (int i = n - 1; i > 0; i--) {

        // Move current root to end
        int temp = arr[0]; 
        arr[0] = arr[i];
        arr[i] = temp;

        // Call max heapify on the reduced heap
        heapify(arr, i, 0);
    }
}

// A utility function to print array of size n
void printArray(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

// Driver's code
int main() {
    int arr[] = {9, 4, 3, 8, 10, 2, 5}; 
    int n = sizeof(arr) / sizeof(arr[0]);

    heapSort(arr, n);

    printf("Sorted array is \n");
    printArray(arr, n);
    return 0;
}
```

#### 기본 개념

힙 정렬은 기본적으로 **배열을 힙으로 만들고, 거기서 다시 정렬을 수행하는 알고리즘**이다.

힙 선택 기준
- **오름차순 정렬**: max heap 사용
- **내림차순 정렬**: min heap 사용

이렇게 하는 이유는 **루트 노드(최댓값 또는 최솟값)를 추출하여 배열의 맨 뒤부터 차례대로 배치**하기 때문이다.

#### 시간 복잡도

**전체 시간 복잡도: O(n log n)**

#### 알고리즘 구성

`heapSort()` 함수는 **두 개의 주요 과정**으로 나뉜다:

1. 힙 구성 (Build Heap)
	- **과정**: 무작위 배열을 max heap으로 변환
	- **시간 복잡도**: O(n)
	- **설명**: 배열의 상태에 따라 다르지만, 최악의 경우에도 거의 n에 근접하는 횟수의 연산이 이루어진다.

2. 정렬 수행 (Sorting)
	- **과정**: 힙에서 차례대로 최댓값을 추출하여 정렬
	- **시간 복잡도**: O(n log n)
	- **설명**:
	    - 각 노드 n개에 대해 heapify 연산을 수행
	    - heapify 자체는 O(log n)의 시간이 걸림
	    - 이를 n번 반복하므로 O(n log n)

#### 전체 시간 복잡도 계산

```
O(n) + O(n log n) = O(n log n)
```

힙 구성 단계의 O(n)은 정렬 단계의 O(n log n)에 흡수되어 전체적으로 **O(n log n)** 이 된다.

#### 핵심 포인트

1. **안정적인 성능**: 최선, 평균, 최악의 경우 모두 $O(n log n)$
2. **제자리 정렬**: 추가 메모리 공간이 거의 필요하지 않음 $(O(1))$
3. **비안정 정렬**: 동일한 값을 가진 원소들의 상대적 순서가 보장되지 않음


### 퀵 정렬 Quick Sort

병합정렬과 비슷하다, 퀵 정렬도 Divide-Conquer 알고리즘의 예시.
Divide : 배열을 특정 인덱스 q를 기준으로 `[low ... q-1]`과 `[q+1 ... high]`로 나뉜다.
Conquer : 위의 두 부속 배열들에서 다시 퀵 정렬을 재귀 호출

1. 배열 안의 항목이 1개 이하라면 반환한다.
2. 배열 안의 한 항목을 Pivot으로 선택한다. (보통 배열의 제일 왼쪽/오른쪽 항목)
3. 배열을 pivot보다 작은 항목들과 큰 항목인 두 부분으로 정리해서 나눈다.
4. 원본 배열의 두 반쪽에 대해 알고리즘을 재귀적으로 반복한다.

```c
// partition function
int partition(int arr[], int low, int high) {
    
    // Choose the pivot
    int pivot = arr[high];
    
    // Index of smaller element and indicates 
    // the right position of pivot found so far
    int i = low - 1;

    // Traverse arr[low..high] and move all smaller
    // elements to the left side. Elements from low to 
    // i are smaller after every iteration
    for (int j = low; j <= high - 1; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(&arr[i], &arr[j]);
        }
    }
    
    // Move pivot after smaller elements and
    // return its position
    swap(&arr[i + 1], &arr[high]);  
    return i + 1;
}

// The QuickSort function implementation
void quickSort(int arr[], int low, int high) {
    if (low < high) {
        
        // pi is the partition return index of pivot
        int pi = partition(arr, low, high);

        // recursion calls for smaller elements
        // and greater or equals elements
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

void swap(int* a, int* b) {
    int t = *a;
    *a = *b;
    *b = t;
}
```

퀵 정렬의 문제는 계속해서 피벗을 최대값 혹은 최솟값으로 잡아서 두 개의 부분 배열로 나누어지지 않게 되는 상황이 발생하면 시간 복잡도가 $O(n^2)$라는 것.
