# item 54) null이 아닌, 빈 컬렉션이나 배열을 반환하라

## null을 반환할 경우

* 컬렉션이 비었을(empty) 때 null을 반환하는 경우, 클라이언트에서는 null 상황을 처리하는 코드를 추가로 작성해야 한다.
* 컬렉션이나 배열같은 컨테이너가 빌 때 null 반환하는 메서드를 사용할 때에 아래와 같이 방어 코드가 없다면 오류가 발생 할 수 있다.

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheese != null && cheeses.conteatins(cheese.STILTON))
    ...
```

* null 대신 빈 컨테이너를 반환하면, 오류 발생을 막고 클라이언트 코드를 간결하게 만들 수 있다.

## 빈 컨테이너 할당 시 비용 문제

* 성능 분석을 통해(item67) 빈 컨테이너 할당이 성능 저하의 주범이라고 확인되지 않는 한, 크게 신경 쓸 수준의 성능 저하는 없다.
* 빈 컬렉션과 배열을 새로 할당 안하고도 Collections에 있는 emptyList, emptySet, emptyMap 등의 메서드를 통해 불변 컬렉션을 반환할 수 있다.

```java
public List<Cheese> getCheese(){
		return cheeseInStock.isEmpty() ?
		Colleciton.emptyList() : new ArrayList<>(cheesesInStock);
}
```

* 배열을 반환하는 경우에도 null 대신 길이가 0인 배열 반환하기
* static final으로 길이가 0인 불변 객체를 선언해두고 공유해도 안전하다.

```java
public static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);// 1) 직접 길이 0 배열을 할당해 사용
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);// 2) 미리 만들어둔 0 배열을 사용
}
```

* 그렇다고 아래 예제와 같이 미리 toArray에 넘기는 배열을 할당하는 것은 성능이 떨어질 수 있다. 그냥 위 예시처럼 사용하도록 한다.

```java
public static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[cheesesInStock.size()];
}
```
