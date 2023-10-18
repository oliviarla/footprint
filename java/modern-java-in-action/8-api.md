---
description: 자바 컬렉션의 개선된 API들을 살펴보자.
---

# 8장: 컬렉션 API 개선

## 컬렉션 팩토리

* 자바 9에 새로 추가되었다.
* of 팩토리 메서드를 통해 작은 리스트, 집합, 맵을 쉽게 만들 수 있다.
*   필요성

    * 기존에는 Arrays.asList() 라는 팩토리 메서드를 사용해 작은 리스트를 만들었다. 고정 크기의 리스트이기 때문에 요소를 추가하려면 UnsupportedOperationException이 발생한다.

    ```java
    List<String> friends = Arrays.asList("Raphael", "Olivia");
    ```

    * Set, Map의 경우 몇개의 요소만 받아 객체를 생성하는 팩토리 메서드가 제공되지 않았다.

### 리스트 팩토리

* List.of 팩토리 메서드를 이용하면 간단하게 리스트를 만들 수 있다.
* 변경할 수 없는 리스트로 반환하기 때문에 add()나 set() 메서드를 사용할 수 없다.

```java
List<String> friends = List.of("Raphael", "Olivia","Thibaut");
```

* 데이터 처리 형식 설정이나 데이터 변환이 필요한 경우 스트림 API를 사용해 리스트를 만들어야 한다.

### 집합 팩토리

* Set.of() 메서드를 사용하면 되며, 리스트와 유사하다.

```java
Set<String> friends = Set.of("Raphael", "Olivia","Thibaut");
```

### 맵 팩토리

* 키와 값을 번갈아 인자로 입력해 맵을 만들 수 있다.

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```

* 10개 이상의 키와 값 쌍을 가진 맵을 만들때는 Map.Entry\<K, V> 객체를 인수로 받는 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다.

```java
Map<String, Integer> ageOfFrieds = Map.ofEntries(
  entry("Raphael", 30),
  entry("Olivia", 25),
  entry("Thibaut", 26));
```

## 리스트와 집합 처리

* **List, Set** 인터페이스에 **호출한 기존 컬렉션 자체를 변경하는 메서드**들이 추가되었다.
  * `removeIf`: 프레디케이트를 만족하는 요소를 제거한다.
  * `replaceAll`: UnaryOperator 함수를 이용해 요소를 바꾼다. 리스트에서 사용할 수 있다.
  * `sort` : 리스트를 정렬한다.

### removeIf

* removeIf 메서드는 삭제할 요소를 가리키는 Predicate를 인수를 받아 해당 조건이 참인 요소를 제거해준다.
*   필요성

    * for-each 문을 사용할 경우 Iterator 객체로 컬렉션을 순회한다.
    * 이로 인해 **Iterator 객체의 상태와 컬렉션 객체의 상태가 동기화되지 않을 수 있다**.
    * 즉, 아래와 같이 컬렉션에 remove나 add 연산을 할 경우 ConcurrentModificationException이 발생할 수 있다.

    ```java
    for(Transaction transaction : transactions) {
      if(Character.isDigit(transaction.getReferanceCode().charAt(0))) {
        transaction.remove(transaction);
      }
    }
    ```

    * 위 코드를 removeIf 메서드를 활용하여 iterate할 필요조차 없이 간단하게 개선할 수 있다.

    ```java
    transactions.removeIf(transaction -> 
      Character.isDigit(transaction.getReferenceCode().charAt(0)));
    ```

### replaceAll

* 리스트의 각 요소를 새로운 요소로 바꿀 수 있다.
*   필요성

    * 첫 문자만 대문자로 변경하고 싶을 때 아래와 같이 작성할 수 있지만, 기존 컬렉션을 변경하는 것이 아니라 새로운 문자열 컬렉션을 만든다.

    ```java
    referenceCode.stream().map(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1))
        .collect(Collectors.toList())
        .forEach(System.out::println);
    ```

    * 자바 8의 replaceAll 메서드를 사용하면 간단하게 기존 컬렉션을 변경할 수 있다.

    ```java
    referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) 
                                                             + code.subString(1));
    ```

## 맵 처리

* Map 인터페이스는 자주 사용하는 패턴을 간편하게 사용할 수 있고 버그를 방지하기 위해 다양한 디폴트 메서드를 제공한다.

### forEach

* 맵에서 키와 값을 반복하면서 확인하는 작업은 번거롭다.
* `Map.Entry<K, V>` 의 반복자를 이용해 맵의 항목 집합을 반복할 수 있다.

```java
for(Map.Entry<String, Integer> entry : ageOfFriends.entitySet()) {
  String friends = entry.getKey();
  Integer age = entry.getValue();
  System.out.println(friend + " is " + age + " years old");
}
```

* forEach 메서드에 키와 인수를 받는 `BiConsumer`형식의 함수를 입력받아 간단하게 구현할 수도 있다.

```java
ageOfFrends.forEach((friends, age) -> 
                          System.out.println(friend + " is " + age + " years old"));
```

### 정렬

* `Entry.comparingByKey`, `Entry.comparingByValue` 유틸리티 메서드를 활용해 맵의 항목을 키 또는 값을 기준으로 정렬할 수 있다.

```java
Map<String, String> favouriteMovies = Map.ofEntries(
  entry("Raphael", "Star Wars"),
  entry("Cristina", "Matrix"),
  entry("Olivia", "James Bond"));
  
favouriteMovies.entrySet().stream()
  .sorted(Entry.ComparingByKey())
  .forEachOrdered(System.out::println);
```

### **getOutDefault**

* 첫 번째 인수로 키를, 두 번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환하는 메서드이다.
* 요청결과를 확인하지 않아도 NullPointerException 문제를 해결할 수 있다.

```java
Map<String, String> favouriteMovies = Map.ofEntries(
  entry("Raphael", "Star Wars"),
  entry("Olivia", "James Bond"));

System.out.println(favorieMovies.getOrDefault("Olivia", "Matrix")); //James Bond 출력
System.out.println(favorieMovies.getOrDefault("Thibaut", "Matrix")); // 기본값인 Matrix 출력
```

### **계산 패턴**

* 맵에 키가 존재하는지 여부에 따라 동작을 수행하고자 할 때에 다음 메서드를 활용할 수 있다.
  * computeIfAbsent : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가한다.
  * computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
  * compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

```java
lines.forEach(line ->
    dataToHash.computeIfAbsent(line, this::calculateDigest));

private byte[] calculateDigest(String key) {
    return messageDigest.digest(key.getBytes(StandardCharset.UTF_8));
}
```

```java
friendsToMovies.computeIfAbsent("Oliviarla", name -> new ArrayList<>())
               .add("Elemental"); // 위 라인에서 생성된 리스트에 요소 추가
```

### 삭제 패턴

* 제공된 키에 해당하는 엔트리를 제거하는 remove 메서드 외에, **키에 매핑되는 값이 특정한 값일 때만 항목을 제거**하는 remoe 메서드도 제공한다.

```java
default boolean remove(Object key, Object value) { ... }
```

### 교체 패턴

* replaceAll: 입력받은 BiFunction을 적용한 결과로 각 항목의 값을 교체한다. List의 replaceAll과 비슷한 동작을 수행한다.
* replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 존재한다.

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars"),
favouriteMovies.put("Olivia", "James Bond");

favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

### 병합

*   putAll

    * 하나의 맵에 다른 맵의 모든 요소를 추가하려 할 때 이 메서드를 사용하면 된다.

    ```java
    Map<String, String> everyone = newHashMap<>(family);
    everyone.putAll(friends);
    ```
*   merge

    * 두 맵을 병합할 때 중복된 키를 처리하는 로직을 담은 BiFunction을 인수로 받아 충돌을 해결할 수 있다.

    ```java
    Map<String, String> everyone = newHashMap<>(family);
    friends.forEach((k, v) -> everyone.merge(k, v, 
                                    (movie1, movie2) -> movie1 + " & " + movie2));
    ```

    * 널값과 관련된 복잡한 상황도 처리한다. 값이 null이면 항목을 제거하거나 매핑함수의 값으로 대치한다.
    * 다음 코드는 키의 반환값이 null이라면 두번째 인자를 기본값으로 저장해두고, 키의 반환값이 null이 아니라면 세번째 인자 함수의 실행 결과를 값으로 저장한다.

    ```java
    moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
    ```

## 개선된 ConcurrentHashMap

* 동시성을 위한 HashMap이다.
* 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.
* 동기화된 Hashtable에 비해 읽기 쓰기 연산이 월등하다.

### 리듀스와 검색

* 다음의 연산을 지원한다.
  * forEach : 각 (키,값) 쌍에 주어진 액션을 실행
  * reduce : 모든 (키,값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
  * search : 널이 아닌 값을 반환할 때까지 각 (키,값) 쌍에 함수를 적용
* 키 , 값으로 연산하거나, 키만으로 연산하거나, 값만으로 연산하거나, Map.entry 객체로 연산할 수 있도록 다양한 메서드를 지원해준다.
  * 키, 값으로 연산 (forEach, reduce, search)
  * 키로 연산 (forEachKey, reduceKeys, searchKeys)
  * 값으로 연산 (forEachValue, reduceValues, searchValues)
  * Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

#### 병렬성 기준값(threshhold)

* 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다.
* 기준값이 1일때는 공통 스레드 풀을 이용해 병렬성을 극대화하며, 기준값이 Long.MAX\_VALUE일때는 한 개의 스레드로 연산을 실행한다.
* 소프트웨어 아키텍처가 고급 수준의 자원 활용 최적화를 사용하지 않는다면 기준값 규칙을 따르는 게 좋다.
* 아래 코드는 reduceValues 메서드를 이용해 맵의 최댓값을 찾는다.

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, 
                                                                  Long::max));
```

### 계수

* 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다.
* size 대신 mappingCount 를 사용해야 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있다.

### 집합뷰

* 집합 뷰로 반환하는 keySet 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 집합을 바꾸면 맵도 영향을 받는다.
* newkeySet 메서드를 이용하면 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.
