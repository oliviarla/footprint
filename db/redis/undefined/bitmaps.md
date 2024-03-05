# Bitmaps

## 개념

* 아주 간단하게 말하면, bit를 저장하는 배열이라고 보면 된다. 이 배열의 **인덱스**를 잘 활용하면 메모리를 효율적으로 사용할 수 있다.
* 예를 들어 유저 ID와 같이 순차적으로 증가하는 값을 저장해야 하는 상황에서 Set 자료구조에 ID 목록을 저장하는 것보다 bitmap에 플래그를 두는 것이 훨씬 메모리 공간을 절감할 수 있다.
* 각 bit의 저장 위치는 offset에 따라 정해진다.
* Bitmap은 2차원 배열로 구성되어 있어 내부적으로 저장되는 위치는 아래 공식에 의해 정해진다.
  * `Offset = Y coordinate * max_width_of_map + X coordinate`
* offset은 배열의 인덱스와 비슷한 개념이라고 보면 되며, `0 ~ 2^32-1` 사이의 값이어야 한다.
* value는 배열의 인덱스에 저장되는 값이라고 보면 되며, bit를 저장하기 때문에 0 또는 1로 설정 가능하다.

<figure><img src="../../../.gitbook/assets/Untitled (2).png" alt=""><figcaption></figcaption></figure>

## 활용

### **설문 조사 대상 유저인지 확인하기**

* 예를 들어 설문조사 대상인 user id를 담아두는 Set이 있다면, 메모리 사용량은 `4 bytes * user id 개수` 가 된다.
* 이를 bitmap을 사용하도록 변경한다면, `1 byte * user id 개수` 만큼만 메모리를 사용하면서 Set과 동일한 역할을 수행할 수 있다.

<figure><img src="../../../.gitbook/assets/Untitled 1.png" alt=""><figcaption></figcaption></figure>

### 일일 방문자 수 구하기

* 일일 방문자 수를 구할 때 방문한 user id의 인덱스에 해당하는 값을 1로 갱신하도록 한다.
* `BITCOUNT` 명령을 사용하여 비트 수를 구할 수 있으므로, 방문한 유저의 수만 조회할 수 있다.
* 이를 통해 천만명의 유저가 있더라도 1.2MB 정도의 크기만 사용할 수 있다.
* 하지만 UUID등을 PK로 사용하는 경우라면 bitmap 사용이 불가능하다.

<figure><img src="../../../.gitbook/assets/Untitled 2.png" alt=""><figcaption></figcaption></figure>

## 사용법

* bitmap에 데이터를 추가할 수 있다.

```
SETBIT key offset value
```

* 저장된 bit를 조회할 수 있다.

```
GETBIT key offset
```

#### 출처

[https://blog.dramancompany.com/2022/10/유저-목록을-redis-bitmap-구조로-저장하여-메모리-절약하기/](https://blog.dramancompany.com/2022/10/%EC%9C%A0%EC%A0%80-%EB%AA%A9%EB%A1%9D%EC%9D%84-redis-bitmap-%EA%B5%AC%EC%A1%B0%EB%A1%9C-%EC%A0%80%EC%9E%A5%ED%95%98%EC%97%AC-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%A0%88%EC%95%BD%ED%95%98%EA%B8%B0/)
