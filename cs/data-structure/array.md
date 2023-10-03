---
description: 인접한 메모리에 원소들을 일렬로 저장하는 데이터 구조
---

# Array

## 특징

* 인접한 메모리 위치에 동일한 유형의 여러 데이터들을 함께 저장하는 방식
* 배열의 첫 번째 요소는 일반적으로 배열 이름이다.
* 배열의 첫 번째 요소가 있는 메모리 위치에 오프셋을 추가하여 쉽게 각 요소의 위치를 ​​계산할 수 있다.
* 배열 생성 시 적절한 크기의 메모리 공간에 정적으로 할당해주기 때문에, 해당 메모리 공간의 다음 부분이 비어있는지 확신할 수 없다. 따라서 배열의 크기를 변경할 수 없도록 한다.
* 데이터가 연속적으로 저장되어 있으므로 공간 지역성 덕분에 캐시 메모리에 저장 시 높은 Cache Hit Rate를 가질 수 있다.
* 삽입과 삭제의 경우 O(N)의 시간 복잡도를 갖는다.
* 원소에 접근할 때는 O(1)의 시간 복잡도를 갖는다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 자바의 배열

* 리스트와 달리 primitive 타입, 객체 타입 모두 배열로 선언 가능하다.
  * ex) `Integer[] arr = new Integer[100];`

#### 배열 기본 동작

```java
// 선언
// 직접 원소를 넣은 채로 선언하거나, 배열의 길이만 먼저 선언할 수 있다.
int[] odds = {1, 3, 5, 7, 9};
String[] weeks = new String[7];

// 값 설정
weeks[1] = "Monday";

// 값 접근 (인덱스 기반만 가능)
System.out.println(weeks[5]);

// 배열 길이 조회
System.out.println(odds.length);
```

#### 배열 정렬

```java
// 오름차순 정렬
Arrays.sort(arr);
System.out.println(Arrays.toString(arr)); // [0,1,2,3,4]

// 내림차순 정렬
Arrays.sort(arr, Collections.reverseOrder());
```

#### 배열 비교

```java
boolean isSame = Arrays.equals(arr1, arr2);
```



#### 참고

[https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94-%EB%B0%B0%EC%97%B4Array-%EB%AC%B8%EB%B2%95-%EC%9D%91%EC%9A%A9-%EC%B4%9D%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94-%EB%B0%B0%EC%97%B4Array-%EB%AC%B8%EB%B2%95-%EC%9D%91%EC%9A%A9-%EC%B4%9D%EC%A0%95%EB%A6%AC)
