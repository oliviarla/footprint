# 정렬

## 안정 정렬 (Stable Sort)

* 동일한 값을 가진 원소들의 경우 **원래 순서가 유지**되는 정렬 방식
* **Stable**: Merge, Bubble, Insertion, Counting, Radix
* **Unstable**: Quick, Heap, Selection

## Comparison-based Sorting

#### Bubble Sort

* 인접한 두 원소를 비교하여 큰 값을 뒤로 보내며 반복 정렬하는 방식
  * 0 \~ n 범위를 인접한 두 원소를 비교하며 가장 큰 값을 맨 마지막 인덱스에 저장한다.
  * 0 \~ n-1 범위를 인접한 두 원소를 비교하며 두번째로 큰 값을 맨 마지막 인덱스에 저장한다.
  * 이 과정을 0 \~ 1 범위까지 진행한다.

```python
for front_index in range(0, len(arr) - 1):
    for index in range(0, len(arr) - 1 - front_index):
        if arr[index] > arr[index + 1]:
            arr[index], arr[index + 1] = arr[index + 1], arr[index]
```

* 시간복잡도
  * 평균: O(n²)
  * 최악: O(n²)
* 공간복잡도: O(1)
* Stable Sort이다.
* 구현이 매우 쉽다.
* 속도가 매우 느려 실제로 사용되지 않는다.

***

#### Selection Sort

* 가장 작은(또는 큰) 값을 선택해 앞(또는 뒤)으로 보내는 방식

```python
for i in range(1, len(arr)):
    key = arr[i]
    j = i - 1
    while j >= 0 and arr[j] > key:
        arr[j + 1] = arr[j]
        j -= 1
    arr[j + 1] = key
```

* 시간복잡도
  * 평균/최악: O(n²)
* 공간복잡도: O(1)
* Unstable Sort이다.
* 장점: 교환 횟수가 최소
* 단점: 전체적으로 비효율적
* 사용 예: 메모리 교환 비용이 매우 비쌀 때 고려 가능

***

#### Insertion Sort

* 필요한 위치에 원소를 삽입해 정렬하는 방식
* 시간복잡도
  * 평균/최악: O(n²)
  * 최선(거의 정렬되어 있는 상태): O(n)
* 공간복잡도: O(1)
* Stable Sort이다.
* 구현이 쉽고, 거의 정렬된 데이터나 작은 배열의 경우 매우 효율적이다.
* 인접한 메모리와의 비교를 반복하기에 참조 지역성의 원리를 잘 만족한다.
* 큰 데이터를 정렬할 때는 비효율적이다.

***

#### Merge Sort

* Divide & Conquer 기법을 통해 배열을 분할 후 병합하면서 정렬하는 방식
* 시간복잡도
  * 평균/최악: O(n log n)
* 공간복잡도: O(n)
* 안정성: Stable
* 장점: 성능 보장, 외부 정렬에 적합
* 단점: 추가 메모리 필요
* 사용 예: 파일 정렬, 대규모 데이터 외부 정렬

***

#### Quick Sort

* 기준값(pivot)을 기준으로 분할 정렬하는 방식
* 시간복잡도
  * 평균: O(n log n)
  * 최악: O(n²) (pivot 선택 실패 시)
* 공간복잡도: O(log n)
* 안정성: Unstable
* 장점: 평균적으로 가장 빠른 비교 정렬
* 단점: pivot 선택에 따라 성능 편차 큼
* 사용 예: C/C++ 표준 라이브러리 정렬 기반

***

#### Heap Sort

* 완전 이진 트리 기반 힙에서 가장 큰 값을 반복 추출하는 방식
* 시간복잡도
  * 평균/최악: O(n log n)
* 공간복잡도: O(1)
* 안정성: Unstable
* 장점: 메모리 절약, 시간복잡도 안정적
* 단점: 실제 성능은 퀵소트보다 느림
* 사용 예: 메모리를 아껴야 하는 환경

***

#### Tree Sort

* 이진 탐색 트리에 데이터를 삽입 후 In-order traversal하여 정렬하는 방식
* 시간복잡도
  * 평균: O(n log n)
  * 최악(BST 불균형): O(n²)
* 공간복잡도: O(n)
* 안정성: Stable (balanced tree일 때)
* 장점: 중복 처리에 유연
* 단점: Tree balancing 필요
* 사용 예: Self-balanced BST 기반 정렬 시

#### Tim Sort

* Insertion sort와 Merge sort를 결합한 알고리즘

## Non-comparison Sorting

#### Counting Sort

* 값의 범위를 기반으로 카운트를 누적하여 정렬하는 방식
* 시간복잡도: O(n + k)\
  (n: 데이터 개수, k: 값의 범위)
* 공간복잡도: O(n + k)
* 안정성: Stable
* 장점: 범위 제한 시 매우 빠름
* 단점: 범위가 넓으면 비효율
* 사용 예: 시험 점수 정렬, 해시 가능한 정수 데이터

***

#### Radix Sort&#x20;

* 정수를 자릿수(LSB/MSB) 기준으로 Counting Sort 반복 적용하는 방식
* 시간복잡도: O(d(n + k))\
  (d: 자릿수 수)
* 공간복잡도: O(n + k)
* 안정성: Stable
* 장점: 정수 정렬에 매우 빠름
* 단점: k와 d에 따라 비효율 가능
* 사용 예: 주민번호/학번/전화번호 정렬

***

#### Bucket Sort

* 버킷으로 나눠 각각 정렬 후 합치는 방식
* 시간복잡도: 평균 O(n + k)
* 공간복잡도: O(n + k)
* 안정성: Stable
* 장점: 데이터 분포가 균일할 때 효율적임
* 단점: 분포가 치우치면 성능 저하
* 사용 예: 실수 데이터 정렬



**출처**

* [https://d2.naver.com/helloworld/0315536](https://d2.naver.com/helloworld/0315536)
